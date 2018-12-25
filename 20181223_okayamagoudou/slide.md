Cookbook 'App Engine'
===

### 合同勉強会in大都会岡山

###### 2018/12/22
###### Masayuki Hayashida ( [@hayashiki](https://github.com/hayashiki) )

---

# <img src="https://avatars1.githubusercontent.com/u/3266316?s=460&v=4" width="50px"> 自己紹介


|key|value|
|---|-----|
|名前|林田 賢行 |
|Name|Hayashida Masayuki|
|Github|[@hayashiki](https://github.com/hayashiki)|
|所属|BULB株式会社 VR/機械学習/IoT|
|コミュニティ|GCPUGオーガナイザー|
|その他|フルリモートワーカー|

---

# 話すこと

- ## AppEngine(GAE) Overview
- ## 構成パターン
	- ### API Endpoint
	- ### Webhook
	- ### SPA
	- ### With Firebase

---

# AppEngineとは

GoogleCloud（GCP）の提供するPaaSサービス :+1:

```
・Webアプリケーションに設定ファイルを書くだけですぐにデプロイ可能
・サーバメンテナンス不要
・オートスケール、圧倒的スピンアップの速さ
・独自ドメインが割り振られる
```

---

## で、大事なポイントが

---

# 無料で運用できる枠がわりとある

- 個人でのサービス開発
- スタートアップでの新規開発
- 検証用プロタイプ
- 社内の業務ハックツールなどなど

---

# AppEngine ランタイム環境

## 対応言語 [Java, Python, PHP, Go, Nodejs]

###### 例としてGoであれば、GAE/Goといった表記がされることが多い


```
正確にいうと、FlexibleEnvironmentは自由度の高いGAE環境がある
例えばRubyといった言語サポートも増えるのだが、無料枠がないこともあり、
今回の話の対象外としている
```

---

# Hello World

#### app.yaml

```
runtime: go111
```

#### app.go

```
func main() {
  http.HandleFunc("/", indexHandler)
  port := os.Getenv("PORT")
  log.Printf("Listening on port %s", port)
  log.Fatal(http.ListenAndServe(fmt.Sprintf(":%s", port), nil))
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
  if r.URL.Path != "/" {
    http.NotFound(w, r)
    return
  }
  fmt.Fprint(w, "Hello, World!")
}
```

---

## デプロイ

``$ gcloud app deploy``


![](https://i.imgur.com/djqo6wQ.png)


---

# その他機能

|Name|Description|
|---|-----|
|Datastore|NoSQLデータベース|
|TaskQueue(Cloud Tasks)|非同期処理ができるタスクキュー|
|Cron(Cloud Sheduler)|スケジューラ|
|GCPのAPI|GCP全般|

<br/>
Logging,Debug機能も実はすごいんだが割愛・・・


---

# Demo

![](https://i.imgur.com/PEnPUrI.png)

---

# Demo解説

![](https://i.imgur.com/9QhzLg8.png)

---

# 構成パターン

---

# APIエンドポイント パターン

- API開発効率が非常にいい

```
バージョン,サービス単位でエンドポイントを付与してデプロイが可能
カジュアルにエンドポイントがたてられる
```

---

## エンドポイント命名

https://[VERSION]-dot-[SERVICE]-dot-[PROJECT].appspot.com

---

# エンドポイント例

``$ gcloud app deploy --version feature``

```
target service:  [base]
target version:  [feature]
target url:      [https://base-dot-goa-api01.appspot.com]
```


``$ gcloud app deploy --version master``

```
target service:  [base]
target version:  [master]
target url:      [https://base-dot-goa-api01.appspot.com]
```
---

# Master

![](https://i.imgur.com/yRUhBPO.png)

---

# Feature

![](https://i.imgur.com/3YDbr8m.png)

---

# なにがいいの？

![](https://i.imgur.com/qeeT97o.png)


- ## 個人,開発機,モバイル向け,Mock用
- ## ロールバック
- ## トラフィック分割（A/B Test）

---

# API開発はgoa使おう

<img src="https://i.imgur.com/cwX6vXr.png">

```
goaはGoで実装されたDSL
API雛形(コード)やSwagger(ドキュメント)を生成してくれる
すぐにAppEngineにのせてデプロイすることができる。
```
---

# まとめ

```
サーバ構築、運用しなくていいとかもう色々とびこえている感あり
コアなビジネスロジック書くところに集中できる環境
```

---

## Webhook / Bot パターン

- ### Webhookを介してWebサービス同士をつなぐ
- ### イベント駆動でなく、定期処理も行うBot

---

# Lineだとこんな設定画面

![](https://i.imgur.com/pYiECKD.png)

---

# Slackだとこんな画面

 ![](https://i.imgur.com/ghjsbkC.png)

---

# Githubだとこんな画面

![](https://i.imgur.com/7Q89uDx.png)

---

# GithubでのBot・Webhook例

<img src="https://i.imgur.com/STCiQHC.png" width="600px">


### - PRの緊急重要ラベルを定期監視してSlackにPostするBotマン

# <img src="https://i.imgur.com/udoaouj.png" width="300px">


### - GithubとSlackのコメントのメンションをSyncさせる

---

# まとめ

```
スケジューラ機能をつかった定期Botとしても実装しやすい
FaaSでよく使われる使い方である
アプリケーション単位でサービス管理したい場合に有効
```

---

# SPA パターン

### HTML,CSS,JavaScriptといったStaticファイルも一緒にデプロイ可能だがリッチなUI/UXがほしい


---

# Vuejsが手っ取り早い

index.htmlにVuejsのCDNを組み込めば簡単にSPAが実現可能 :smiley:

![](https://i.imgur.com/2BHuXUX.png)

---

# index.htmlにCDNをくみこむ

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Okayama!</title>
    <script src="http://unpkg.com/vue/dist/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  </head>
```
---

# データバインディング

```
var app = new Vue({
  el: '#app',
  data: {
    tasks: [],
    newTask: "",
  },
  created: function() {
    axios.get('/tasks')
      .then((response) => {
        this.tasks = response.data.items || []
  })},
  methods: {
    addTask: function(task) {
      let params = new URLSearchParams()
      params.append('body', this.newTask)
      axios.post('/tasks', params)
        .then((response) => {
          ...
```
--- 

## React, AngularでもSPA使いたい

##### Webpack build ビルド先をAppEngineのビルド先に出力

<img src="https://i.imgur.com/wV8UCze.png" width="330px">

---

# まとめ

## BackendとFrontend、両方をデプロイすることが可能

---

# With Firebase パターン

![](https://i.imgur.com/nmliRX9.png)

---

# FirebaseAuthだけお借りするパターン

![](https://i.imgur.com/mhYhF63.png)

```
AppEngineのアプリをFirebaseで認証する
Firebaseが使用するユーザーIDを借りて
それ以外の永続化データはDatastoreで管理する
```

---

## さきほどのデモは実は認証しないと投稿できないようになっている

![](https://i.imgur.com/1maABPV.png)


---

# FirebaseのFirestore

## FirebaseのDataBase Firestoreを利用する

![](https://i.imgur.com/pBsPLtP.png)

---

# AppEngine + Firebase

## お互いに得意な領域で役割をわける

- Frontend側はUIを作り込むトコロ
- バッチ処理、複雑なビジネスロジックを含む処理

例：1日1回バッチで数万件のアイテムを一括で情報更新、
XXば場合なレポートを作成し、全文検索Index更新し、
さらにビジネスロジックが云々・・・

---

・フロントエンドで、こみいった非同期処理、並行処理など辛み :cold_sweat:

## ならばAppEnging/GOでGoroutinesをつかう

![](https://i.imgur.com/S3AObRg.png)

---

# まとめ

## FirebaseとAppEngineをハイブリッドに使おう

---

# DDD!!!（どんどんデプロイ） :+1: :+1: :+1:

- ## 岡山AppEngineハンズオン

https://gcpug-okayama.connpass.com

- ## 岡山Go勉強会

https://okayamago.connpass.com/
https://okayamago.connpass.com/event/112138/


<img src="https://i.imgur.com/4uzABiB.png" width="350px">

---

# ref

- Next Currency-GAEGo
https://speakerdeck.com/sonatard/next-currency-gaego

- GolangのgoaでAPIをデザインしよう
https://tikasan.hatenablog.com/entry/2017/05/08/190000

- App Engine アプリのユーザーを Firebase で認証する
https://cloudplatform-jp.googleblog.com/2016/10/app-engine-firebase.html

