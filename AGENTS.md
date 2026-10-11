# AGENTS.md — AI エージェント向け開発ガイド

このリポジトリで作業する AI コーディングエージェント (Claude Code / Gemini CLI / Antigravity など) 向けのガイドです。これまでの開発 (issue #1〜#37) で得た知見を整理しています。

## プロジェクト概要

KataGo を Google Colaboratory 上で GPU 実行し、クリック操作できる碁盤 GUI で検討するための単一ノートブック (`katago-on-colab.ipynb`)。ビルドシステムやテストスイートはなく、成果物はノートブックそのものです。

- 対象環境: Colab の GPU ランタイム (L4 基準。T4 でも動作確認済み #35)
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
- **stderr も専用スレッドで読んでログに残す**。共有ライブラリ不足やモデルのバージョン非対応などの起動エラーは stderr にしか出ない (KataGo 自身の `gtp_logs/` にも残らない)。起動待ち中にプロセスが終了したら即座に失敗とし、stderr の末尾をエラーに含める (#32)
- **手順の移動は redo スタックで行う**。UNDO / 10手戻すで戻した手を `(色, 座標)` で積み、10手進めるで打ち直す。戻した手と同じ手を打ったときだけ続きを残し、それ以外の着手・新規対局・SGF 読み込みでは破棄する。複数手まとめて動かすときも showboard は最後に1回だけ取る (#37)
- JS への状態受け渡しは base64 経由 (`invokeFunction` の戻り値は text/plain の repr で、手動アンエスケープはクォートやバックスラッシュで壊れる) (#8)

## KataGo の設定・モデル

- `gtp.cfg` は Colab L4 基準。`numSearchThreads = 12` / `nnMaxBatchSize = 6` は L4 + tf3-b11c768 での `benchmark -tune` 実測の推奨値 (#30。旧 b28 では 24 だった #11)。v1.18 の `-tune` はバッチサイズ (スレッド数の半分) も推奨する。GPU やモデル構造が変わったら再計測を促す。T4 + tf3 では 5 スレッドが推奨されたが、12 スレッド / バッチ 6 の既定値との速度差は 2% 未満 (約 265〜270 visits/s。L4 の約 4 割) なので既定値は共通のままにしている (#35)
- `benchmark -tune` は推奨スレッド数を**表示するだけ**で cfg は書き換えない。対話的な自動生成は `genconfig` (#5)
- ポンダリングは無効 (`ponderingEnabled = false`)。常時解析モードが GPU を使うため不要
- モデル名の規則: `kata1-<構造>-s<学習ステップ数>-d<データ行数>`。構造は `b28c512nbt` (28ブロック・512チャンネル・nested bottleneck の CNN) や `tf3-b11c768` (トランスフォーマー) など。新しいモデルは `s12501M` のように M 単位で略記される。同一構造なら s / d が大きいほど新しく強い (#16)
- 2026-08 以降のメイン系統はトランスフォーマー (`tf3-b11c768`)。**トランスフォーマーは KataGo v1.17.0 以降でないと読めない**。cuDNN 8.9.7 ビルドはトランスフォーマーで大幅に遅いので cuDNN 9 ビルドを使う (#30)
- katagotraining.org の API (`/api/networks/`、新しい順) は Python 既定の User-Agent を 403 で弾く。**明示的な User-Agent ヘッダを付ける** (#16)
- モデルや Colab 環境が変わったときの確認手順 (#30):
  1. モデル名の構造部分 (`b28c512nbt`、`tf3-…` など) を見て、新しい接頭辞なら読める KataGo のバージョンをリリースノートで調べる
  2. Colab の CUDA を `nvidia-smi` で確認し、KataGo ビルド名 (`cuda13.2-…`) のメジャーバージョンを合わせる
  3. `katago version` で共有ライブラリが揃っているか確認する
  4. `benchmark -tune` をやり直し、`numSearchThreads` と `nnMaxBatchSize` を反映する

## Colab の制約

- ランタイムタイプ (GPU 種別) はノートブック内から変更できない (ユーザーの UI 操作が必要)
- コンピューティングユニットの消費は**ランタイム接続時間ベース**で、GPU 使用率には依存しない
- **Colab のベースイメージは 2026-10 時点で Ubuntu 24.04 / CUDA 13.0 / cuDNN 9 (CUDA 13 用) がプリインストール**。CUDA 12 系ライブラリは無く、apt の `libcudnn8` も存在しない。KataGo は `cuda13.x-cudnn9.x` ビルドを使う (cuda12 ビルドは `libcublas.so.12` が無くて起動しない) (#30)
- KataGo の Linux バイナリは AppImage (static-pie) なので `ldd` では依存を確認できない。`katago version` を実行すれば共有ライブラリ不足がエラーとして出る
- colab-mcp の `run_code_cell` は長時間セルでクライアント側タイムアウトするが、セル自体は実行が継続する。`get_cells` の出力ポーリングで完了を確認する

## Colab 実機検証の進め方

- 検証は空のノートブック (colab-mcp の接続先) に検証用セルを足して行う。リポジトリのノートブック本体は書き換えない
- push 済みのセルは GitHub raw から取得して `get_ipython().run_cell(src)` で実行できる。未 push の変更は、ローカルでセルのソースを zlib + base64 にしてセルに貼り、展開後に sha256 をローカルと照合してから実行する
- `get_cells` は `cellIndexStart` / `cellIndexEnd` で範囲を絞る。大きなセルを含むノートブック全体を取ると出力が巨大になる
- `!pkill -f katago` は、その `!` 行を実行するシェル自身のコマンドラインにも一致して先に自分を殺してしまい、KataGo が残る。終了処理セルと同じ psutil 方式で止める
- モデル形式のエラーはバックエンドに関係なく同じ処理で出る。GPU ライブラリが無い環境でも CPU 版 (eigen / eigenavx2) のバイナリで再現できる (#32)
- GUI の描画 (canvas) は colab-mcp からは見えない。Colab で `api_get_state()` の戻り値 (base64) を取得し、ローカルで `HTML_UI` に `google.colab.kernel.invokeFunction` のモックを付けて `google-chrome --headless=new --screenshot` で描画を確認できる (#37)
- **検証が終わったら検証用セルをすべて削除する** (public リポジトリのため、検証用のコードや出力を残さない)
- 最後にランタイムを切断する。`from google.colab import runtime; runtime.unassign()` で「接続解除して削除」になり、コンピューティングユニットの消費が止まる。切断の確認のためにセルを実行すると新しいランタイムに再接続してしまうので、確認はユーザーに画面表示で行ってもらう
