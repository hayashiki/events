# Firebaseの実践的な話

---

# はなすこと

- Firebase Authentication
- Firebase Firestrore 多対設計

---

# Firebase Authentication

---

## User登録について

### Firebase Authentication

こんなやつ

![](https://i.imgur.com/A9ZZa2J.png)

---

## User管理の視点で話す

ユーザ一覧ってどう取得するの？

---

## Firebaseで提供はしている

例：

```
function listAllUsers(nextPageToken) {
  // List batch of users, 1000 at a time.
  admin.auth().listUsers(1000, nextPageToken)
    .then(function(listUsersResult) {
      listUsersResult.users.forEach(function(userRecord) {
        console.log("user", userRecord.toJSON());
      });
      if (listUsersResult.pageToken) {
        // List next batch of users.
        listAllUsers(listUsersResult.pageToken)
      }
    })
}
```

---

## 設計でまようところ

実践では、ユーザはモデルとしてカスタムな属性がほしい場合がほとんど
で、業務ロジックがはいってくる場合の対応がしづらい。
例：課金ユーザのフラグだとか、退会ユーザだとか。

---

## 解決方法

FirestoreにUserInfoテーブルをつくる（名前は任意）
Firebase Authenticationで作成されるUidをキーとする

---

## ではどうやってFirestoreに格納する？

---

## コールバック

// googleログインをクリックしたときに動作する関数例

```
handleLogin = async () => {

  // google login
  const authData = await this.database.googleLogin();

  // userInfoに格納
  firestore.doc(`userInfo/${authData.uid}`).set(
    {
      id: authData.uid,
      state: 'active',
      email: authData.email
    }
  )
}
```
---


## Firebase Authenticationトリガー

https://firebase.google.com/docs/functions/auth-events?hl=ja

```
exports.createUser = functions.firestore
  .document("users/{userId}")
  .onCreate((snap, context) => {
    let { email, name } = snap.data()
    let uid = context.params.userId

    return admin
      .auth()
      .createUser({
        uid,
        email,
        displayName: name,
      })
  })
```
---

## どちらがよいかはコンテクストによってかわる

難しいところだが、
userInfoへ登録だけあればコールバックで良いと考えている

---

## どちらがよいかはコンテクストによってかわる

・Email送信
・デフォルトのグループやコミュニティにはいる処理があるドメイン
・Push通知をおくる
　例：デフォルトのコミュニティに自動ではいって、そのメンバたちに通知する的な

---

## まとめ

処理が複雑であれば、Functionによせるほうがよいが、
できるだけクライアントに仕事をしてもらいたいので、シンプルであれば
クライアント側の非同期処理で対応する

---


## LTする上での前提

以下のテーブルをイメージしてほしい

- users
  - posts
  - tags

ユーザがいて、ユーザがPostできて
そのPostにタグをつけることができるアプリケーションを
イメージしてほしい

---

# Firebase Firestrore 多対設計

```
firestore-root
    |
    --- posts (collections)
    |     |
    |     --- postId (document):0cCIGH6Dunw2TcLubvzS
    |            |
    |            --- title: "Question Title"
    |            |
    |            --- body: "Question Title"
    |            |
    |            --- tags (subCollections)
    |                  |
    |                  --- tagId: "0cCIGH6Dunw2TcLubvzS"(document)
    |                  |
    |                  --- tagId (document)
    |
    --- tags (collections)
    |     |
    |     --- tagId (document): 0cCIGH6Dunw2TcLubvzS
    |            |
    |            --- name: "JS"
    |            |
    |            --- posts (subCollections)
    |                  |
    |                  --- postsId: "0cCIGH6Dunw2TcLubvzS"(document)
    |                  |
    |                  --- postsId (document)
```

---

# 相互参照パターン

postsとtagsテーブルにお互いに参照する形をとる

RDB的には、PostTagなのを使って交差（中間）テーブルを用意することが多いと思う。
Firebaseはモバイル重視なので、一度に表示する件数は少ない場合が多いので、
N+1的な性能問題は発生しづらい。

---

## 一括更新で管理

(ちょっとサンプルが適当すぎたのであとでかきなおします)

```
const batch = firestore.batch();
batch.set(firestore.collection('posts').doc(id), {
  tags: [:tag_id],
});
batch.set(firestore.collection('tags').doc(id), {
  posts: [:post_id]
});
batch.commit();
```

---

## Algoliaパターン

[Algolia](https://www.algolia.com/doc/)とは、全文検索のSaaSサービス

---

## algolia のインデックス登録

firestoreにデータが登録更新されたら、functions経由でalgoliaへインデックス登録するようにします。
Postが登録されたら、Index登録できるようにする

```
exports.onPostCreated = functions.firestore.document('posts/{documentId}')
	.onCreate((snap, context) => {
		const post = snap.data();		
		const index = algolia.initIndex("tags")
		index.saveObject(post)
	})
```

---

## 検索

react前提にはなるが、react-instantsearchというライブラリにて検索する
（ここの詳細はまた今度発表します。そんな方法もあるんだくらいで認識してください）


```
<InstantSearch
  appId="app_id"
  apiKey="hohohohohohohooho"
  indexName="tags"
>
  <AutoCompleteWithData />
<InstantSearch/>
```
