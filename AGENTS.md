# AGENTS.md — AI エージェント向け開発ガイド

このリポジトリで作業する AI コーディングエージェント (Claude Code / Gemini CLI / Antigravity など) 向けのガイドです。これまでの開発 (issue #1〜#27) で得た知見を整理しています。

## プロジェクト概要

KataGo を Google Colaboratory 上で GPU 実行し、クリック操作できる碁盤 GUI で検討するための単一ノートブック (`katago-on-colab.ipynb`)。ビルドシステムやテストスイートはなく、成果物はノートブックそのものです。

- 対象環境: Colab の GPU ランタイム (L4 基準。T4 でも動作)
- 構成: `README.md` (日英ミラー構成) / `katago-on-colab.ipynb` / `img/` (スクリーンショット) / `LICENSE` (MIT)

## 開発ワークフロー (必須)

1. 変更は必ず先に GitHub issue を作る。本文は日本語で「現状 / 問題 / 対応方針」を記載する
2. コミットは変更単位で分ける。メッセージは英語で、本文に `Fixes #N` を含める (push で issue が自動クローズされる)
3. 可能なら Colab 実機で検証し (公式 colab-mcp が使える)、検証結果を issue にコメントとして記録する
4. このリポジトリのコミットは noreply メール名義で行う (リポジトリローカルの git config 設定済み)

## ノートブック編集の注意

- `.ipynb` は JSON (indent=2、`source` は行のリスト、日本語は生バイト)。編集後は JSON 妥当性と、コードセルの構文 (`!` 行を `pass` に置換して `compile()`) を確認する
- 実行出力 (outputs) はコミットしない
- **セルマジック (`%%writefile` 等) は必ずセルの1行目に置く**。`# @title` と併用すると UsageError で動かない (#9)
- 画像は base64 埋め込み禁止 (GitHub のノートブックビューアは data URI を描画しない)。`img/` に置いて raw.githubusercontent.com の絶対 URL で参照する。Colab で開いた場合も表示され、リポジトリが public であることが前提 (#7, #13)
- **GUI セルの `HTML_UI` 内の JS にバックスラッシュを書くときは Python のエスケープ解釈を考慮する**。`HTML_UI` は通常の三重クォート文字列なので、JS に `\n` を届けるにはソース上 `\\n` と書く。`\n` のままだと実改行としてブラウザに届き、JS の文字列リテラルが途中で切れて `<script>` 全体が構文エラーになる (ボタンが全滅する)。JS の構文チェックはセルのソーステキストではなく、Python 評価後の `HTML_UI` (`ast.parse` で代入を探して `ast.literal_eval`) から `<script>` 部を取り出して `node --check` にかける (#27)

## KataGo / GTP 連携の設計 (GUI セル)

過去に踏んだ罠がすべて現在の設計に反映されています。以下を壊さないこと:

- **stdout は専用スレッド + `queue.Queue` で読む**。`select()` とバッファ付き `readline()` の併用は、readline が先読みした行を fd から消してしまい、以降の読み取りが常に1レスポンス遅れる (#10)
- **showboard は専用リーダー (`read_showboard`) で読む**。GTP 応答は `=` 行が先頭に来るため、`=` で打ち切る `send_and_wait` では盤面本体を読めない。盤面の最下段 (行番号 1) まで読み切る (#10)
- **常時解析モード**: `kata-analyze interval 30 rootInfo true` を回しっぱなしにする。`get_state` (GUI の1.5秒ポーリング) では GTP コマンドを送らない (送ると解析が止まる)。盤面は着手時のみ showboard で取得してキャッシュする (#12)
- **コマンド送信前に必ず `stop_analysis()` を呼ぶ**。解析報告が1件も出る前 (interval の 0.3 秒以内) にコマンドを送ると、kata-analyze の閉じ応答が `=` 単体で出力され、直後のコマンドの成否判定を誤る (#17)
- GTP コマンドの成否は応答が `=` で始まるかで判定し、**成功時のみ** Python 側の状態 (手番・手数・履歴) を更新する (#1)
- 起動待ちは固定 sleep ではなく `version` コマンドの応答待ち (GTP エンジンは準備完了までコマンドに応答しない) (#2)
- JS への状態受け渡しは base64 経由 (`invokeFunction` の戻り値は text/plain の repr で、手動アンエスケープはクォートやバックスラッシュで壊れる) (#8)

## KataGo の設定・モデル

- `gtp.cfg` は Colab L4 基準。`numSearchThreads = 24` は L4 + b28 モデルでの `benchmark -tune` 実測値 (#11)。GPU が変わったら再計測を促す
- `benchmark -tune` は推奨スレッド数を**表示するだけ**で cfg は書き換えない。対話的な自動生成は `genconfig` (#5)
- ポンダリングは無効 (`ponderingEnabled = false`)。常時解析モードが GPU を使うため不要
- モデル名の規則: `kata1-<構造>-s<学習ステップ数>-d<データ行数>`。構造は `b28c512nbt` (28ブロック・512チャンネル・nested bottleneck) など。同一構造なら s / d が大きいほど新しく強い。現在のメイン系統は b28c512nbt (#16)
- katagotraining.org の API (`/api/networks/`、新しい順) は Python 既定の User-Agent を 403 で弾く。**明示的な User-Agent ヘッダを付ける** (#16)

## Colab の制約

- ランタイムタイプ (GPU 種別) はノートブック内から変更できない (ユーザーの UI 操作が必要)
- コンピューティングユニットの消費は**ランタイム接続時間ベース**で、GPU 使用率には依存しない
- colab-mcp の `run_code_cell` は長時間セルでクライアント側タイムアウトするが、セル自体は実行が継続する。`get_cells` の出力ポーリングで完了を確認する
