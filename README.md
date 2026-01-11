# katago-on-colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/axyojp/katago-on-colab/blob/main/katago-on-colab.ipynb)

KataGo を Google Colaboratory で簡単に動かすためのノートブックです。

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
基本的には上から下まで順番にセルを実行することで利用できるはずです。



### 開発について
* あくまで検討で使えるツールとして作っているため、シンプルな機能しか実装していません。
* Vibe Coding が世の中に浸透したおかげで作ることができました。
* Colab の制約や便利な機能をうまく組み合わせる必要があり、試行錯誤を一緒にしてくれる AI がとても役に立ちました。
* 今回は Gemini や Antigravity を使って開発を行ってます。
* KataGo の設定はあまり詳しくないので改善があれば教えてください。
* 機能要望はベストエフォートで対応します。


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


### About Development
* Since this tool is intended for analysis, only simple features are implemented.
* This project was made possible thanks to the spread of Vibe Coding.
* Navigating Colab's constraints and features required trial and error, and the AI partner was extremely helpful.
* Development was conducted using Gemini and Antigravity.
* I am not an expert on KataGo settings, so please let me know if there are any improvements.
* Feature requests will be handled on a best-effort basis.


### 謝辞 / Acknowledgments
本プロジェクトは、David J. Wu 氏によって開発された素晴らしい囲碁 AI である KataGo を利用しています。著者に深く敬意を表します。
また、KataGo が MIT ライセンスで公開されていることに倣い、本プロジェクトも MIT ライセンスを採用しています。

This project makes use of KataGo, an amazing Go AI developed by David J. Wu. We express our deepest respect to the author.
Following KataGo's release under the MIT License, this project also adopts the MIT License.
