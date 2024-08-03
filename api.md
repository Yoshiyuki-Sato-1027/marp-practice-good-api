---
marp: true
title: 良いAPI設計について
theme: gaia
---

# 良い API 設計について

---

# 目次

1. [エンドポイント設計](#エンドポイント設計)
2. [レスポンスデータ設計](#レスポンスデータ設計)
3. [エラーの表現](#エラーの表現)
4. [CORS](#CORS)

---

## 1.エンドポイント設計

- **覚えやすく直感的**
- **複数形の名詞**
  ・集合を表すため
  ・URI はリソースを意味するので名刺
- **単語の連結はハイフン（-）**
  - Google がハイフンは単語のつなぎとみなすが、アンダースコアは単に無視してひと続きの単語とみなしてしまうから
  - そもそもつなげるのを避ける

```
popular_users →　users/popular
```

---

- **短く入力しやすい URI**
- **人間が読んで理解できる URI**
- **大文字小文字が混在していない URI**
- **改造しやすい（Hackable な）URI**
- **サーバ側のアーキテクチャが反映されていない URI**
- **ルールが統一された URI**

---

## パスとクエリパラメータの使い分け

### パスを使う場合

- **一意なリソースを表す情報**
  - URI はリソースを表すもの
- **省略不可能な情報**
  - オプションで検索に必要なパラメータ

---

# 2.レスポンスデータ設計

- **JSON か XML か**
- **JSON が推奨される理由**
  - シンプルでサイズが小さい
  - JavaScript との相性が良い
- **データの階層**
  Google の JSON Style Guide
  「なるべくフラットにしたほうがよいけど、階層構造を持ったほうがわかりやすい場合もあるよね」

---

```json
階層的に表す場合
{
  “id”: 3342124,
  “message”: “Hi!“,
  “sender”: {
    “id”: 3456,
    “name”: “Taro Yamada”
  },
  “receiver”: {
    “id”: 12912,
    “name”: “Kenji Suzuki”

フラットに表す場合
{
  “id”: 3342124,
  “message”: “Hi!“,
  “sender_id”: 3456,
  “sender_name”: “Taro Yamada”,
  “receiver_id”: 12912,
  “receiver_name”: “Kenji Suzuki”
}
```

---

### 最上位は配列かオブジェクトか

```json
[
   {
      “id”: 234342,
      “name”: “Taro Tanaka”,
   },
   {
```

```json
{
 “friends”: [
     {
        “id”: 234342,
        “name”: “Taro Tanaka”
     },
     {
```

---

**JSON インジェクションのリスク低減のため、オブジェクトが推奨される**

---

## 3.エラーの表現

- **HTTP を理解したステータスコードを選ぶ**
- **エラー詳細メッセージを渡す**

twitter の例

```json
{
  “errors”: [
    {
      “message”: “Bad Authentication data”,
      “code”: 215
    }
  ]
}
```

---

**開発者用メッセージとユーザー用メッセージを渡す手法**

```json
{
  “error”: {
    “developerMessage”: “...開発者向けエラーメッセージ...“,
    “userMessage”: “...ユーザー向けエラーメッセージ...“,
    “code”: 2013,
    “info”: “http://docs.example.com/api/v1/authentication”
  }
}
```

---

### 4.CORS (Cross-Origin Resource Sharing)

同一生成元ポリシー(同じ生成元（オリジン）からの読み込みのみを許可するというポリシー)
→ 同じ生成元(オリジン)以外でもアクセスできるようにするのが CORS

オリジン・・・ URI 中のスキーム（http や https など）、ホスト（api.example.com など）、ポート番号の組み合わせで判断されます。

---

### CORS の流れ

- **クライアント側 から Origin リクエストヘッダーを送る**

```http
GET /resource HTTP/1.1
Host: api.example.com
Origin: http://example.com
```

---

### サーバー側

- **アクセスを許可する生成元の一覧を保持している**
- **Origin ヘッダーを受け取り、上記に含まれているかチェック**

→ **含まれていない場合**
**403 エラー**

→ **含まれている場合**

**Access-Control-Allow-Origin というレスポンスヘッダに Origin リクエストヘッダと同じ生成元を入れて返す**

```http
Access-Control-Allow-Origin: http://example.com
```

---

### すべての生成元からアクセスを許可する場合

```http
Access-Control-Allow-Origin: *
```

例
GitHub

### プリフライトリクエストとは

別オリジンにアクセスしようとしたときに、OPTIONS メソッドを送り、リクエストが受け入れられるかをチェックする
