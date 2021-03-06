# CloudLogger引継ぎ書類2017

***
## 1. 前書き
僭越ながら，自分への備忘録も兼ねて，引継ぎ書類的なものを書かせていただきます．

## 2. 概要
機体には各種センサーが搭載されており，また表示器として使われるタブレットからもGPSや機体の角度が取得できる．これらのデータを一度サーバー上へアップロードし，TF時にリアルタイムに取得して設計者にフィードバックするのが，CloudLoggerの役割である．

また，コンテスト本番では，最終的に機体は水没し，タブレット内部に保存されているログもロストする(可能性がある)．そこでCloudLoggerを用いることで，確実にログを手に入れようという目論見もある．

## 3. システムの構成
CloudLoggerシステムは主に2つのセクションに分かれている．**(これを読む前に"4.プログラム言語について"を読んだほうがよいかも)**
### 1. タブレット(使用言語:`java`)
機体の各種センサーの値はメイン基板を通してタブレットに送られ，データ送信用のオブジェクト`CloudLoggerService`に格納される．それらのデータは，専用のスレッド`CloudLoggerSendThread`において
，タブレットのデータ通信回線を通じ，所定のフォーマットでサーバーにPOSTで送信される．フォーマットは，例として，`csv/test3.csv`に2つのcsvデータを送信する場合は
```test3.csv:
3,296417,0,228,4,34.47590354024991,133.48898750826285,286,...(省略)
296768,-1,226,5,34.47590510480562,133.48898724810005,287,...(省略)
```
と送信する．冒頭の`3,`で書き込むファイル名を指定している．
### 2. サーバー(使用言語:`php`,`javascript`,`html`)
#### 2-1. フォルダ構成
　|-`writer.php` ...POSTで送信されたデータを`csv/test?.csv`と`lastline.csv`に書き込む  
　|-`lastline.csv`...最新のデータが上書きされていく．  
　|-`line.php`...`lastline.csv`からデータを取得する．  
　|-`watcher.html`...`line.php`からのデータを表示する．  
　|-`output.php`...`~./output.php?filename=test?.csv&lineN=N`で`test?.csv`からN 行目以降を取得する．  
　|-`app.js`...`output.php`を用いて，`csv/test?.csv`のデータをグラフ化する．  
　|-`graph.html`...`app.js`で作成されるグラフを表示する．  
　|-`graphContest.html`...`graph.html`のコンテスト本番用．  
　|-`controller.php`...コントローラーとは名ばかり．`csv`フォルダ以下の`test?.csv`の内容をすべて消去する．  
　|-`csv`...csvファイルが入っている．  
　　|-`test1.csv`  
　　|-`test2.csv`  
　　　.  
　　　.  
#### 2-2. 説明
サーバーには3つの役割がある．
1. タブレットからPOSTで送られてきたデータを，`writer.php`で，ログ用のcsvファイル`csv/test?.csv`(?は数字)と，リアルタイム表示用のcsvファイル`lastline.csv`に保存する．
2. 現在の機体の情報を`lastline.csv`から`line.php`で取得し，`watcher.html`でリアルタイムに表示する．
3. csvフォルダに蓄積されるcsvデータを`output.php`で取得し，データを`app.js`でグラフ化し，`graph.html`に表示する．

## 4．プログラミング言語について
### 1.java
言わずとしれたオブジェクト指向プログラム言語．"Write once, run anywhere" をスローガンにしているように，どのような環境，プラットフォームでも動くよう設計された．そのような機能を実現するために，プラットフォーム,OSごとにJVM(java virtual machine)が用意され，javaのコンパイルはその仮想マシンで実行出来るjavaバイトコードを生成する．  
その能力が買われ(?)，androidのアプリ開発にも用いられている．わが電装班ではパイロットへのデータ表示用にandroidタブレットのアプリを用いている． 詳細は本宮に聞いてね．
最近Kotlinという言語でも開発できるようになったらしい．  
ファイルの拡張子は`.java`である．
### 2.HTML(とXML)
プログラム言語ではない．HyperText Markup Languageの略で，ハイパーテキストを記述する(マークアップ)言語という意味．ハイパーテキストとは，Wikipediaによれは"複数のテキストを相互に関連付け，結び付ける仕組み"らしい．たとえばwebサイトでは，平文だけでなく，ほかのページへのリンクや画像，動画などが配置されているが，それらを配置，結びつけ，表示するのがHTMLである．  
例えば，`photo.jpeg`という画像を表示したい場合は
```
<img src="photo.jpeg">
```
のように記述する．またshootingStartsというサブタイトルをページに追加したいときは
```
<h1>shootingStarts</h>
```
のように記述することで，shootingStartsがサブタイトルとして表示される．
`<h1>`はタグと呼ばれ，これで平文をサンドイッチすることでその文になにかしらの効力を与える．  
世間ではもちろんwebサイトの表示に使われ，webサイト作成においては避けては通れない．  
ちなみに同じマークアップ言語であるXML(Extensible Markup Language)はAndoridアプリの見た目を構成するのに用いられる．  
ファイルの拡張子は`.html`または`.htm`である．
### 3.javascript
javaと名前が似ているが，まったく別者．javascriptが名前をパクっただけ．  
webサイトの表示は確かにhtmlだけで事足りるが，htmlはプログラム言語ではないので，たとえば繰り返しや条件分岐などが出来ない(サイトのボタンが押されたら何々するなど)．そこで登場するのがjavascriptである．javascriptを用いることでユーザーの操作に基づいた表示ができる．  
しかし，あくまでjavascriptが実行されるのはユーザーのブラウザ，PCであり，例えば，サーバー上のファイルを書き換えたり，またログイン処理などもできない.  
今回のCloudLoggerでは機体のデータをリアルタイムに表示したり，csvのログデータを取得し，グラフ化するときに使っている．  
ファイルの拡張子は`.js`

### 4.PHP
スクリプト言語の一種．"PHP is HyperText Processor"といわれるように，ハイパーテキスト(html)を"動的に"生成するための言語．webサーバー上で実行される．というかそれ以外で用いられる場面を見たことがない．  
javascriptと違いサーバー上で実行され，状況に応じたhtmlを生成し，ユーザーに届ける役割をもつ．またサーバー上のファイルの操作もできる．  
今回のcloudLoggerでは中心的な役割を担い，タブレットからのデータを受け取り，csvファイルに書き込んだり，逆にcsvファイルからデータを取り出したりしている．  
このように，サーバー上で動くプログラムはphp以外にもCGIと言われる仕組みがあり，perlやrubyが使われる．ただ，CGIが動くレンタルサーバーは大体有料なので，今回はphpが無料で動かせるxdomainというレンタルサーバーを用いた．  
ファイルの拡張子は`.php`．

***
