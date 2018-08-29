## 概要
IBM CloudのNode-RED上でTwitterから特定キーワードでTweetを収集し、IBM Watson NLCを利用してクラス分類した結果をグラフ表示します。
また、クラス分類したTweetの内訳を表示することができます。


***(デモ)***
- Tweetの内容をNLCによって分類した結果のカウント数を1分単位の折れ線グラフで表示
- 分類結果の累計カウント数・比率を円グラフで表示
- 分類したTweetの内訳を特定の日、時、分を指定して表示(日のみ、日時のみ指定も可)


## 利用条件
- **IBM Cloud account** を持っていること(IBM Watson NLC、Node-RED用)
- **電話番号認証済みのTwitter account** を持っていること(Tweet収集用)

## 利用開始手順
1. IBM Cloudで、Node-RED Starterを作成  
アプリ名、ホスト名、プランを任意で設定し作成します。  
※作成には5分程度かかることがあります。

2. Node-REDにIBM Watson NLCを接続  
接続タブから「新規に接続」を選択して、新規にNLCを作成するか、「既存に接続」を選択して既存のNLCを選択します。

3. Node-REDの初期設定  
Node-REDのアプリURLにアクセスし、画面の指示に従ってエディタ画面で使うユーザ名、パスワードなどを設定します。

4. 「node-red-dashboard」を追加  
フローエディタ画面右上のメニューから「パレットの管理」をクリックし、「ノードを追加」タブで「node-red-dashboard」を検索し、「node-red-dashboard」の「ノードを追加」ボタンをクリックします。

5. フローの読み込み  
フローエディタ画面右上のメニューから「読み込み」->「クリップボード」とクリックし、テキストエリアに「aistarter_nodered_twitter.json」の内容をコピーし、貼り付けます。

6. Cloudantの初期設定  
Node-REDのBluemix Cloud Foundry アプリ管理コンソールの接続タブからCloudantの資格情報を表示し、urlをコピーします。  
Node-REDフローエディタ画面の「Cloudant初期設定」タブで「Cloudant設定」ノードをダブルクリックします。  
「global.cloudant.url」の値に先ほどコピーしたurlを貼り付けます。  
「global.cloudant.dbname」の値に任意の英数字を入力し、「完了」ボタンをクリックします。  
※「global.cloudant.dbname」の値はTwitterから収集したデータと、それの分類結果を格納するデータベース名となります。

![cloudant_credentials1](https://github.com/softbank-developer/twitter_analyzer_on_nodered/blob/master/readme_images/cloudant_credentials1.png)  
![cloudant_credentials2](https://github.com/softbank-developer/twitter_analyzer_on_nodered/blob/master/readme_images/cloudant_credentials2.png)

7. Twitterの設定  
https://apps.twitter.com/  
にアクセスし、アプリを選択します。  
アプリが無い場合は、「Create New App」ボタンをクリックし、必要項目を入力してアプリを作成します。  
アプリ管理画面の「Keys and Access Tokens」タブをクリックし、表示されているConsumer Key(API Key)、Consumer Secret(API Secret)、Access Token、Access Token Secretをメモします。Access Tokenが無い場合は「Create my access token」ボタンをクリックすることで作成されます。  
「SBAnalyzer」タブの「ツイート収集」ノードをダブルクリックし、検索条件を入力します。  
「Twitter ID」横のプルダウンから「新規にtwitter-credentialsを追加」を選択し、横の鉛筆ボタンをクリックします。  
Twitter IDを入力し、それぞれの入力欄にメモしたConsumer Key(API Key)、Consumer Secret(API Secret)、Access Token、Access Token Secretを入力します。

8. Cloudantバインド設定の反映  
「SBAnalyzer」タブの「ツイート情報登録」ノードをダブルクリックし、「Service」に「Node-REDのアプリ名+cloudantNoSQLDB」が表示されることを確認し、「完了」ボタンをクリックします。  
「完了」ボタンをクリックすると、ノードの右上に青い丸がつきます。  
同様に「分類器作成」、「分類器削除」タブの全てのCloudantノードを右上に青い丸がついた状態にします。  
![cloudant_node](https://github.com/softbank-developer/twitter_analyzer_on_nodered/blob/master/readme_images/cloudant_node.png)  
フローエディタ画面右上の「デプロイ」ボタンをクリックします。

9. Database・View作成  
「Cloudant初期設定」タブで「Database作成」「view作成」ノードの左に付いているボタンをクリックします。

10. ポジネガ分類器を作成  
「分類器作成」タブで、「学習データ」ノードにNLCの学習データ形式で学習データをセットします。ポジネガ分類器のclassは「positive,neutral,negative」を想定しています。  
「分類器名セット」ノードに「posinega」をセットします。  
フローエディタ画面右上の「デプロイ」ボタンをクリックします。  
「学習データをセットしてクリック」ノードの左に付いているボタンをクリックします。  
デバッグタブに出力されたclassifier_idをメモします。  
「SBAnalyzer」タブの「NLC:ポジネガ判定」ノードにメモしたclassifier_idを設定し、「完了」ボタンをクリックします。

11. BOT判断分類器を作成  
「分類器作成」タブで、「学習データ」ノードにNLCの学習データ形式で学習データをセットします。BOT判断分類器のclassはBOT判断分類器のclassは「person,others」を想定しています。  
「分類器名セット」ノードに「isbot」をセットします。  
フローエディタ画面右上の「デプロイ」ボタンをクリックします。  
「学習データをセットしてクリック」ノードの左に付いているボタンをクリックします。  
デバッグタブに出力されたclassifier_idをメモします。  
「SBAnalyzer」タブの「NLC:BOT判断」ノードにメモしたclassifier_idを設定し、「完了」ボタンをクリックします。

12. デプロイ  
フローエディタ画面右上の「デプロイ」ボタンをクリックします。


## 使い方
1. グラフを見る  
「(アプリURL)/ui」にアクセスします。  
※起動後すぐにはグラフは表示されません。しばらくお待ちください。  
    - 折れ線グラフは1分単位でTweetの内容をNLCによって分類した結果のカウント数を表示しています。  
    - 円グラフは分類結果の累計カウント数・比率を表示しています。


2. Tweet内訳を見る  
各グラフ下にあるフォームに内訳を見たい日時分を指定します。フォームには日のみ、日・時のみを指定することも可能です。  
    - 指定した時間に呟かれたTweetの分類とTweet内容を表示します。  


## ライセンス

[MIT](https://github.com/softbank-developer/twitter_analyzer_on_nodered/blob/master/LICENSE)

