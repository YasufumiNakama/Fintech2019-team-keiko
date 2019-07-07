# チーム『慶工』
## メンバー 
綱島秀樹　中間康文　枇々木裕太  
## レポート 
FDC_teamkeiko_submit.pdfをご覧ください  
## 概要 
東証1部・2部，マザーズ，JASDAQに上場している日本企業3622社を対象に5～10社のポートフォリオを作成しました  
取り組みは大きく分けて以下の３つになります  
1. 過去の収益率をもとに3622社から30社まで企業数を絞る
2. VAEを用いて投資期間において株価が上昇すると判断された企業をピックアップ　
3. Twitterの感情分析をもとにポートフォリオを作成
# 1. StockSelection 
## 概要  
東証1部・2部，マザーズ，JASDAQに上場している日本企業3622社から30社まで企業数を絞りこみます  
絞り込む基準として，2016年～2018年の3年間の2月の月次収益率に注目し，その平均÷標準偏差の値が大きい30社に絞りこみました  
## 結果  
meigara.Rファイルを実行すると，2016年～2018年の3年間の2月の月次収益率に注目し，その平均÷標準偏差の値が大きい30社がsummary.csvファイルとして出力されます  
# 2. VAE
## 概要  
VAEを用いて投資期間において株価が上昇すると判断された企業をピックアップする  
## 結果  
1. fintech2019.csv  
日系企業の株価の平均/標準偏差をもとに割り出した30社の株価データ  
2. MakeTorchGraph.ipynb(プログラム内のディレクトリ名は要変更)  
Block1 -> グーグルドライブのマウント  
Block2 -> 2019年2月の株価予測用ろうそくチャート制作コマンド  
Block3 -> 2016年~2018年の株価ろうそくチャート制作コマンド  
3. VAE_FDC.ipynb(プログラム内のディレクトリ名は要変更)  
Block1 -> 深層学習フレームワークchainerの環境確認コード  
Block2 -> グーグルドライブのマウント  
Block3-6 -> 訓練可能な形式でろうそくチャートの読み込み  
Block7 -> Block3-6のデータを一度.npz形式で保存すれば読み込むだけで使用可能にするコード  
Block8 -> 訓練データ、検証用データ、テストデータで分離し、訓練可能な形式にするコード  
Block9 -> VAEの本体のコード。ネットワーク、訓練コードを含んでいる  
Block10 -> 2019年2月の株価を予想するためのろうそくチャートの読み込み  
Block11 -> 2019年2月の株価を予想するためのろうそくチャートのstock codeの読み込み  
Block12 -> テストデータの再構成誤差を確認するためのコード  
Block13 -> 2019年2月の予測株価画像の生成コード  
4. results  
model_submitted/FDC_decoder_2750.npz -> VAEのデコーダーの訓練済み重みなど。再構成に必要  
model_submitted/FDC_encoder_2750.npz -> VAEのエンコーダーの訓練済み重みなど。再構成に必要  
model_submitted/log -> 訓練のログ  
train_test/test_tmp.npz -> テストデータ。VAE_FDC.ipynbのBlock7で使用  
train_test/train_tmp.npz -> 訓練データ。VAE_FDC.ipynbのBlock7で使用  
loss.png -> 訓練、検証用データの再構成ロスカーブの画像  
# 3. TwitterSentimentAnalisys  
## 概要  
StockSelectionで挙げられた企業30社をキーワードとして含むTweetのテキスト情報を，以下の条件のもとで取得しました   
①　投資開始日から，過去7日間まで最新のTweetから取得  
②　Tweetの取得件数は最大100件まで  
各企業に対するツイートのテキスト情報をCSVファイルに保存し，MeCabを用いて形態素解析を行いました  
そして既存の感情辞書をもとに感情分析を行いました  
今回の株価予測では，5～10社のポートフォリオを組むという条件があるため，最終的にここで得られた感情指数値をもとにポートフォリオを決定しました  
## 実行結果
実行結果は以下の順序で確認できます  
実行内容については，以下の動作手順と合わせてご確認ください  
1. GetTweetsToCSV.ipynb
2. SentimentScore.ipynb
3. SentimentRank.ipynb  
4. Portfolio.ipynb  
## 実行内容
### API Key
Twitterからデータを取得するためには「Twitter Developer」から「API Key」を取得する必要があります  
個人で取得した「API Key」を公開することはできませんのでコード中では伏せてあります  
そのためコードを実行するには[Twitter Developer](https://developer.twitter.com/)から「API Key」を取得してください  
### 動作手順
1. ./input/keyword.csv  
StockSelectionより検索をかけたい企業名をcsvファイルで用意  

2. GetTweetsToCSV.py  
「API Key」を入力する箇所があるので取得した「API Key」に書き換えてください  
以下のコードを実行すると過去7日間まで遡って最大100件のツイートが取得されtweetsディレクトリ下にCSVファイルで保存されます  
今回は2/10の14:22から過去7日間まで遡って最大100件のツイートを取得しました  
~~~
$ python GetTweetsToCSV.py
~~~
3. SentimentScore.py  
以下のコードを実行すると各ツイートに対するSentimentScoreが計算されsentiment_scoreディレクトリ下にCSVファイルで保存されます  
感情分析を行うにあたっては，東北大学の乾・鈴木研究室が作成した日本語評価極性辞書を用いました（./dictionary下に配置）  
各ツイートに対するツイート感情指数値は，形態素解析した各語句に対するスコアの和を計算することで算出しました  
（ポジティブ：1，ニュートラル：0，ネガティブ：-1）  
~~~
$ python SentimentScore.py
~~~
4. SentimentRank.py  
以下のコードを実行するとキーワードごとにツイートあたりのスコアが計算され降順でソートされsentiment_rankディレクトリ下にCSVファイルで保存されます  
企業に対する企業感情指数値は，企業名をキーワードとして含むツイート全てに対するツイート感情指数値の平均を計算することで算出しました  
~~~
$ python SentimentRank.py
~~~ 
5. Portfolio.ipynb  
VAEの結果とSentimentRankの結果をもとにポートフォリオを組みます  
ここで企業感情指数値を用いて銘柄選択を行うにあたって，以下の条件を採用しました  
①　企業感情指数値が0.5以上  
②　Tweetの取得件数が最低でも10件以上  
またポートフォリオを組むにあたって，以下の条件を採用して最終的なポートフォリオを提出しました      
①　業種で等分  
②　業種が被る場合，その業種の中でさらに等分  
③　全ての単元株が100であったため，予算1000万に収まる範囲で適度に調整  
## 参考
1. https://blog.seiyamaeda.com/11269
2. https://dailytextmining.hatenablog.com/entry/2018/07/12/065500
3. http://www.cl.ecei.tohoku.ac.jp/index.php?Open%20Resources%2FJapanese%20Sentiment%20Polarity%20Dictionary#t6684569
