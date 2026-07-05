# katago-on-colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/axyojp/katago-on-colab/blob/main/katago-on-colab.ipynb)

KataGo を Google Colaboratory で簡単に動かすためのノートブックです。

A notebook for easily running KataGo on Google Colaboratory.

![katago on colab](img/katago-on-colab.png)



## 日本語

### はじめに

KataGo を利用する際に GPU を利用したいことがあると思います。
GPU マシンを用意するのも、難しい設定を行うのも大変な場合があるため、Google Colaboratory で動作するようにしました。

単に動かすだけではなく、GUI で碁盤をクリックして操作できるようにしてあります。

### 事前設定

必ず Runtime の設定で GPU を選択してください。
L4 でも十分です。

### 使い方

基本的には上から下まで順番にセルを実行することで利用できます。

**1. ノートブックを開いて GPU を有効にする**

1. 上の「Open In Colab」バッジからノートブックを開く
2. メニューの「ランタイム」→「ランタイムのタイプを変更」→ ハードウェア アクセラレータで **GPU (L4 推奨)** を選択して保存

**2. セルを上から順に実行する**

| セル | 内容 |
|---|---|
| KataGo Configuration | KataGo 本体とモデルの URL 設定。**既定のままで OK** |
| モデルの選択 (任意) | 最新のネットワーク一覧をドロップダウンから選択。スキップ可 |
| 1. インストール | 依存ライブラリ・KataGo 本体・モデルのダウンロード (数分かかります) |
| 2. 設定ファイルの作成 | gtp.cfg を作成。そのまま実行 |
| 3. ベンチマーク (任意) | 動作確認。スキップ可 |
| 4. 設定の自動調整 (任意) | 最適なスレッド数を計測。表示された推奨値を 2 の `numSearchThreads` に反映して再実行 |
| 5. 実行 | KataGo を起動して GUI を表示 |
| 6. 終了処理 | 使い終わったら実行してプロセスを掃除 |

**3. GUI の使い方**

- **着手**: 碁盤をクリック (黒白交互)。**UNDO** で一手戻し、**PASS** でパス
- **候補手**: 盤上の色付き丸が候補手 (赤いほど有力、数字は手番側の勝率)。右のリストには勝率と探索数 (visits) が表示されます
- **常時解析**: 盤面を表示している間、裏で解析が回り続けます。見ているだけで候補手・勝率がどんどん深まり、現局面の総探索数は「N visits」表示で確認できます
- **GPU 表示**: 情報パネル下部に GPU の種類と使用率。使用率が緑 (30%以上) なら解析が正常に働いています
- **勝率バー**: 白地に黒い部分が黒の勝率です

**4. 終わったら**

「6. 終了処理」セルを実行し、Colab の「ランタイム」→「ランタイムを接続解除して削除」を実行してください (接続中はコンピューティングユニットを消費し続けます)。

### 開発について

* あくまで検討で使えるツールとして作っているため、シンプルな機能しか実装していません。
* Vibe Coding が世の中に浸透したおかげで作ることができました。
* Colab の制約や便利な機能をうまく組み合わせる必要があり、試行錯誤を一緒にしてくれる AI がとても役に立ちました。
* 今回は Gemini や Antigravity、Claude Code を使って開発を行ってます。
* KataGo の設定はあまり詳しくないので改善があれば教えてください。
* 機能要望はベストエフォートで対応します。

### 謝辞

本プロジェクトは、David J. Wu 氏によって開発された素晴らしい囲碁 AI である KataGo を利用しています。著者に深く敬意を表します。
また、KataGo が MIT ライセンスで公開されていることに倣い、本プロジェクトも MIT ライセンスを採用しています。



---



## English

### Introduction

Run KataGo on Google Colaboratory with ease.
This project was created to help those who want to use KataGo with a GPU without the hassle of setting up a GPU machine or complex configurations.

It allows you not only to run KataGo but also to interact with it via a GUI where you can click on the Go board.

### Prerequisites

Please ensure you select **GPU** in the Runtime settings.
An L4 GPU is sufficient.

### Usage

Basically, you can use it by executing the cells in order from top to bottom.

**1. Open the notebook and enable the GPU**

1. Open the notebook via the "Open In Colab" badge above
2. Go to "Runtime" → "Change runtime type" → select **GPU (L4 recommended)** as the hardware accelerator and save

**2. Run the cells from top to bottom**

| Cell | Description |
|---|---|
| KataGo Configuration | URLs for the KataGo release and the model. **Defaults are fine** |
| Select Model (optional) | Pick one of the latest networks from a dropdown. Can be skipped |
| 1. Installation | Downloads dependencies, KataGo, and the model (takes a few minutes) |
| 2. Create Config | Writes gtp.cfg. Just run it |
| 3. Benchmark (optional) | Sanity check. Can be skipped |
| 4. Auto-tune (optional) | Measures the optimal thread count. Apply the suggested value to `numSearchThreads` in step 2 and rerun it |
| 5. Execution | Starts KataGo and shows the GUI |
| 6. Cleanup | Run when you are done to terminate the engine |

**3. Using the GUI**

- **Playing moves**: Click the board (Black and White alternate). **UNDO** takes back a move, **PASS** passes
- **Candidate moves**: Colored circles on the board are candidates (redder = stronger, the number is the side-to-move win rate). The list on the right shows win rates and visit counts
- **Continuous analysis**: While the board is displayed, the engine keeps analyzing in the background. Candidates and win rates keep deepening as you watch; the total visit count is shown as "N visits"
- **GPU indicator**: The bottom of the info panel shows the GPU model and utilization. Green utilization (30%+) means the analysis is working properly
- **Win rate bar**: The black portion of the bar is Black's win rate

**4. When you are done**

Run the "6. Cleanup" cell, then use "Runtime" → "Disconnect and delete runtime" (a connected runtime keeps consuming compute units).

### About Development

* Since this tool is intended for analysis, only simple features are implemented.
* This project was made possible thanks to the spread of Vibe Coding.
* Navigating Colab's constraints and features required trial and error, and the AI partner was extremely helpful.
* Development was conducted using Gemini, Antigravity, and Claude Code.
* I am not an expert on KataGo settings, so please let me know if there are any improvements.
* Feature requests will be handled on a best-effort basis.

### Acknowledgments

This project makes use of KataGo, an amazing Go AI developed by David J. Wu. We express our deepest respect to the author.
Following KataGo's release under the MIT License, this project also adopts the MIT License.
