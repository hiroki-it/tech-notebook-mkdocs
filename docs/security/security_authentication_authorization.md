---
title: 【IT技術の知見】Authenticate（認証）/Authorization（認可）＠セキュリティ
description: Authenticate（認証）/Authorization（認可）＠セキュリティの知見を記録しています。
---

# Authenticate（認証）/Authorization（認可）＠セキュリティ

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. 認証

### 認証とは

通信しているユーザーが誰であるかを特定する方法。

<br>

### HTTP認証

#### ▼ HTTP認証とは

HTTP通信の中で認証を行う認証スキームのこと。リクエストの```authorization```ヘッダーとレスポンスの```WWW-Authenticate```ヘッダーで認証スキームを指定する。認証スキームの種類には、『Basic認証』、『Digest認証』、『Bearer認証』などがある。認証情報の一時的な保存は、ブラウザのWebStoregeで行うため、認証解除（ログアウト）をサーバー側で完全に制御できない。

ℹ️ 参考：

- https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml
- https://architecting.hateblo.jp/entry/2020/03/27/130535
- https://developer.mozilla.org/ja/docs/Web/HTTP/Authentication#authentication_schemes

<br>

### Form認証（Cookieベースの認証）

#### ▼ Form認証とは

認証時に、```Cookie```ヘッダーの値を使用する認証スキームのこと。『Cookieベースの認証』ともいう。ステートフル化を行うため、HTTP認証には属していない。認証情報の一時的な保存は、サーバーのセッションデータで行うため、認証解除（ログアウト）をサーバー側で制御できる。```Cookie```ヘッダーによる送受信では、CSRFの危険性がある。

ℹ️ 参考：

- https://h50146.www5.hpe.com/products/software/security/icewall/iwsoftware/report/pdfs/certification.pdf
- https://auth0.com/docs/sessions/cookies#cookie-based-authentication

#### ▼ セッションIDを使用したForm認証の場合（セッションベース）

セッションIDを```Cookie```ヘッダーに割り当て、リクエストを送信する。

最初、ユーザー作成の段階で、クライアントが認証情報をサーバーに送信する。サーバーは、認証情報をDBに保存する。

```http
POST https://example.com/users

{
    "email_address": "foo@gmail.com",
    "password": "foo"
}
```

次回の認証時に、再びユーザーが認証情報を送信する。

```http
POST https://example.com/foo-form

{
    "email_address": "foo@gmail.com",
    "password": "foo"
}
```

サーバーは、DBの認証情報を照合し、ログインを許可する。サーバーは、セッションIDを作成し、セッションデータに書き込む。

```bash
# セッションデータ
{ sessionid: ***** }
```

レスポンスの```Set-Cookie```ヘッダーを使用して、セッションIDをクライアントに送信する。

```http
200 OK
Set-Cookie: sessionid=<セッションID>
```

サーバーは、セッションIDとユーザーIDを紐付けてサーバー内に保存する。加えて次回のログイン時、クライアントは、リクエストの```Cookie```ヘッダーを使用して、セッションIDをクライアントに送信する。サーバーは、保存されたセッションIDに紐付くユーザーIDから、ユーザーを特定し、ログインを許可する。これにより、改めて認証情報を送信せずに、素早くログインできるようになる。

```http
POST https://example.com/foo-form
cookie: sessionid=<セッションID>
```

認証解除時、サーバーでセッションデータを削除する。

ℹ️ 参考：https://blog.tokumaru.org/2013/02/purpose-and-implementation-of-the-logout-function.html

#### ▼ トークンを使用したForm認証の場合（トークンベース）

トークンを```Cookie```ヘッダーに割り当て、リクエストを送信する。この時のトークンの選択肢として、単なるランダムな文字列やJWTがある。

ℹ️ 参考：https://scrapbox.io/fendo181/JWT(JSON_Web_Token)%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B%E3%80%82

![JWT](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/JWT.png)

#### ▼ ```Cookie```ヘッダーの値のクライアント保持

再利用のため、```Cookie```ヘッダーに割り当てるための値（セッションID、トークン）は、ブラウザを通して、ローカルマシンに有効期限に応じた間だけ保持できる。またはブラウザの設定によって、ブラウザのWebストレージでも保持できる。Chromeの場合は、Cookieストレージに保持される。確認方法については、以下のリンクを参考にせよ。

ℹ️ 参考：

- https://developer.chrome.com/docs/devtools/storage/cookies/
- https://qiita.com/cobachan/items/05fa537a4ffcb189d001

<br>

### APIキー認証

#### ▼ APIキー認証とは

事前にAPIキーとなる文字列を配布し、認証フェースは行わずに認可フェーズのみでユーザーを照合する認証スキームのこと。API GatewayにおけるAPIキー認証については、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/cloud_computing/cloud_computing_aws.html

#### ▼ 照合情報の送信方法

独自ヘッダーとして、```x-api-key```ヘッダーを定義する。これにAPIキーを割り当て、リクエストを送信する。リクエストヘッダへのパラメータの割り当てについては、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_application_collaboration_api_restful.html

```http
GET https://example.com/bar.php
x-api-key: <APIキー>
```

<br>

### PAT：Personal Access Tokenによる認証

#### ▼ PATによる認証

クライアントがPersonal Access Token（個人用アクセストークン）の付与をリクエストし、認証フェースは行わずに認可フェーズのみでユーザーを照合する。```Authorization```ヘッダーにPATを割りあてて、リクエストを送信する。作成時以降、アクセストークンを確認できなくなるため、クライアントがアクセストークンを管理する必要がある。

ℹ️ 参考：https://www.contentful.com/help/personal-access-tokens/

```http
GET https://example.com/bar.php
authorization: <Personal Acccess Token>
```

| サービス例 | トークン名            | 説明                                                         |
| ---------- | --------------------- | ------------------------------------------------------------ |
| GitHub     | Personal access Token | HTTPSを使用して、プライベートリポジトリにリクエストを送信するために必要。HTTPSを使用する場面として、アプリケーションの拡張機能のGitHub連携、リポジトリのパッケージ化、などがある。<br>ℹ️ 参考：https://docs.github.com/ja/github/authenticating-to-github/creating-a-personal-access-token |

<br>

## 01-02. HTTP認証の種類

### Basic認証

#### ▼ Basic認証とは

認証時に、平文のIDとパスワードを使用する認証スキームのこと。

#### ▼ 仕組み

![Basic認証](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/Basic認証.png)


| 役割         | 説明                                                         |
| ------------ | ------------------------------------------------------------ |
| クライアント | リクエスト送信元のアプリケーションのこと。文脈によっては、ブラウザがクライアントである場合とそうでない場合（例：OAuth認証）がある。 |
| ユーザー       | クライアントを使用している人物のこと。                       |
| サーバー       | クライアントからリクエストを受信し、レスポンスを送信するアプリケーションのこと。 |

最初、クライアントは、認証後にアクセスできるWebページのリクエストをサーバーに送信する。

```http
GET https://example.com/foo-form
```

サーバーは、これ拒否し、```401```ステータスで認証領域を設定し、レスポンスを送信する。これにより、認証領域の値をユーザーに示して、ユーザー名とパスワードの入力を求められる。ユーザーに表示するための認証領域には、任意の値を持たせられ、サイト名が設定されることが多い。

```http
401 Unauthorized
WWW-Authenticate: Basic realm="<認証領域>", charaset="UTF-8"
```

『```<ユーザー名>:<パスワード>```』をbase64方式でエンコードした値を```authorization```ヘッダーに割り当て、リクエストを送信する。

```http
POST https://example.com/foo-form
authorization: Basic bG9naW46cGFzc3dvcmQ=
```

サーバーは、ユーザー名とパスワードを照合し、合致していれば、認証後のWebページを返信する。また、認証情報をブラウザのWebストレージに保存する。

```http
200 OK
WWW-Authenticate: Basic realm=""
```

認証の解除時は、誤った認証情報をブラウザに意図的に送信させて認証を失敗させるようにする。

ℹ️ 参考：https://stackoverflow.com/questions/4163122/http-basic-authentication-log-out

```http
POST https://example.com/foo-form/logout
authorization: Basic <誤った認証情報>
```

サーバーは、```401```ステータスでレスポンスを返信し、認証が解除される。

```http
401 Unauthorized
WWW-Authenticate: Basic realm="<認証領域>", charaset="UTF-8"
```

<br>


### Digest認証

#### ▼ Digest認証とは

認証時に、ハッシュ化されたIDとパスワードを使用する認証スキームのこと。

#### ▼ Digest認証の仕組み

```http
200 OK
WWW-Authenticate: Basic realm="<認証領域>", charaset="UTF-8"
```

```http
POST https://example.com/foo-form
authorization: Digest realm="<認証領域>" nonce="<サーバー側が作成した任意の文字列>" algorithm="<ハッシュ関数名>" qoq="auth"
```

<br>

### Bearer認証

#### ▼ Bearer認証とは

認証時に、Bearerトークンを使用する認証スキームのこと。

#### ▼ Bearerトークン（署名なしトークン）とは

単なる文字列で定義されたアクセストークン。Bearer認証にて、トークンとして使用する。署名なしトークンとも呼ばれ、実際に認証された本人か否かを判定する機能は無く、トークンを持っていればそれを本人として認可する。そのため、トークンの文字列が流出してしまわないよう、厳重に管理する必要がある。

ℹ️ 参考：https://openid-foundation-japan.github.io/rfc6750.ja.html#anchor3

#### ▼ Bearer認証の仕組み

指定されたエンドポイントに対して、```POST```リクエストを送信する。この時、```Content-Type```ヘッダーを```application/x-www-form-urlencoded```とする。必要なボディパラメーターはAPIの提供元によって異なる。クライアントID、付与タイプ、などが必要なことが多い。

ℹ️ 参考：

- https://developer.amazon.com/ja/docs/adm/request-access-token.html#request-format
- https://ja.developer.box.com/reference/post-oauth2-token/#request

```http
POST https://example.com/foo
Content-Type: application/x-www-form-urlencoded
    
# ボディ
client_id=*****&grant_type=client_credentials&scope=messaging:push
```

レスポンスボディにBearerトークンを含むレスポンスが返信される。他に、有効期限、権限のスコープ、指定できる認証スキーマ、などが提供されることが多い。

ℹ️ 参考：

- https://developer.amazon.com/ja/docs/adm/request-access-token.html#request-format
- https://ja.developer.box.com/reference/resources/access-token/

```http
200 OK
X-Amzn-RequestId: d917ceac-2245-11e2-a270-0bc161cb589d
Content-Type: application/json

{
  "access_token":"*****",
  "expires_in":3600,
  "scope":"messaging:push",
  "token_type":"Bearer"
}
```

発行されたBearerトークンを指定された認証スキーマで```Authorization```ヘッダーに割り当て、リクエストを送信する。ここでは詳しく言及しないが、BearerトークンをForm認証のように```Cookie```ヘッダーに割り当てることもある。

ℹ️ 参考：

- https://stackoverflow.com/questions/34817617/should-jwt-be-stored-in-localstorage-or-cookie
- https://ja.developer.box.com/reference/post-oauth2-token/#response

```http
POST https://example.com/foo
authorization: Bearer <Bearerトークン>
```

サーバーは、Bearerトークンを照合し、合致していれば、認証後のWebページを返信する。無効なBearerトークンをブラックリストとしてRedis/DBで管理しておく。DBでブラックリストを管理すると、リクエストの度にDBアクセス処理が実行されることなってしまうため、Redisでこれを管理した方が良い。

```http
200 OK
WWW-Authenticate: Bearer realm=""
```

認証の解除時は、Redis/DBでBearerトークンの状態を無効化する。またサーバーは、```401```ステータスでレスポンスを返信し、認証が解除される。

ℹ️ 参考：

- https://stackoverflow.com/questions/21978658/invalidating-json-web-tokens
- https://medium.com/devgorilla/how-to-log-out-when-using-jwt-a8c7823e8a6

```http
401 Unauthorized
WWW-Authenticate: Basic realm="<認証領域>", charaset="UTF-8"
```

#### ▼ 正常系/異常系レスポンス

ℹ️ 参考：https://qiita.com/h_tyokinuhata/items/ab8e0337085997be04b1

成功の場合は、realm属性を空にしたレスポンスを返信する。

```http
200 OK
WWW-Authenticate: Bearer realm=""
```

失敗の場合は、error属性にエラメッセージを割り当てたレスポンスを返信する。

```http
400 Bad Request
WWW-Authenticate: Bearer error="invalid_request"
```

```http
401 Unauthorized
WWW-Authenticate: Bearer realm="token_required"
```

```http
403 Forbidden
WWW-Authenticate: Bearer error="insufficient_scope"
```

#### ▼ ```Authorization```ヘッダーのトークンのクライアント保持

不便ではあるが、```Authorization```ヘッダーは```Cookie```ヘッダーとは異なり、ローカルマシンに保存できない。その代わり、ブラウザの設定によって、ブラウザのWebStorageでも保持できる。Chromeでは、LocalStorage/SessionStorageに保持される。LocalStorageはSessionStorageと比べて保存期間が長いため、XSSの危険性がより高い。これらの確認方法については、以下のリンクを参考にせよ

ℹ️ 参考：

- https://developer.chrome.com/docs/devtools/storage/localstorage/
- https://developer.chrome.com/docs/devtools/storage/sessionstorage/
- https://stackoverflow.com/questions/5523140/html5-local-storage-vs-session-storage

<br>

### OAuth認証

ノート内の[こちら](#02. 認可フェーズ)を参考にせよ。

<br>

### SAML認証

ノート内の[こちら](#02. 認可フェーズ)を参考にせよ。

<br>

## 01-03. 複数の認証の組み合わせ

### Two Step Verification（二段階認証）

#### ▼ Two Step Verificationとは

認証時に、段階的に```2```個の方法を設定し、クライアントを照合する。

| 一段階目の認証例 | 二段階目の認証例 | 説明                                                         | 備考                                         |
| ---------------- | ---------------- | ------------------------------------------------------------ | -------------------------------------------- |
| IDとパスワード   | IDとパスワード   | IDとパスワードによる方法の後、別のIDとパスワードによる方法を設定する。 |                                              |
|                  | 秘密の質問       | IDとパスワードによる方法の後、質問に対してあらかじめ設定した回答による方法を設定する。 |                                              |
|                  | SMS              | IDとパスワードによる方法の後、SMS宛に送信した認証コードによる方法を設定する。 | 異なる要素のため、これは二要素認証でもある。 |
|                  | 指紋             | IDとパスワードによる方法の後、指紋の解析結果による方法を設定する。 | 異なる要素のため、これは二要素認証でもある。 |

<br>

### Two Factor Authorization（二要素認証）

#### ▼ Two Factor Authorizationとは

二段階認証のうちで特に、認証時に異なる要素の方法を使用して、段階的にクライアントを照合すること方法のこと。後述するOAuth認証を組み込んでも良い。

| 一要素目の認証例       | 二要素目の認証例                                             |
| ---------------------- | ------------------------------------------------------------ |
| IDとパスワード（知識） | 事前登録された電話番号のSMSで受信したワンタイムパスワード（所持） |
|                        | 事前登録された電話番号のSMSで受信した認証コード（所持）      |
|                        | OAuth認証（所持）                                            |
|                        | 指紋（生体）                                                 |
| 暗証番号（知識）       | キャッシュカード（所持）                                     |

<br>

## 02. 認可

### 認可とは

認証済みのユーザーに対して、アクセス可能な権限範囲（認可スコープ）を付与する方法。

<br>

### SSO：Single Sign On

#### ▼ SSOとは

Webサイトごとに認証/認可を行うのではなく、特定のWebサイトに認証フェーズを委譲し、認可フェースはWebサイト別に行う。

#### ▼ 認証フェーズと認可フェーズ

![sso](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/sso.jpg)

SSOには、認証フェーズと認可フェーズがあり、```3```個の役割が定義されている。

ℹ️ 参考：https://japan.zdnet.com/article/35126144/

1. クライアントが、HTTPリクエストにIDとパスワードを設定してリクエスト。
2. IdP：Identity Providerが、IDを『認証』し、クライアント側にアクセストークンを発行。
3. クライアントが、HTTPリクエストのヘッダーにアクセストークンを設定してリクエスト。
4. アクセストークンが『認可』されれば、API側がデータをレスポンスする。

| 役割              | 説明                                                         | 例                                          |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------- |
| APIクライアント   | APIに対して、リクエストを送信したいサーバーのこと。          | Ouath認証の仕組みにおけるクライアント。     |
| Identity Provider | トークンを作成するサーバーのこと。                           | Ouath認証の仕組みにおける認可サーバー。     |
| APIサーバー       | クライアントに対して、リソースのレスポンスを送信するサーバーのこと。 | Ouath認証の仕組みにおけるリソースサーバー。 |

#### ▼ ステータスコード

認証フェーズにて、誤ったトークンが発行されたことを表現したい場合、```401```ステータスを使用する。認可フェーズにて、正しいトークンが発行されたが、トークンの所有者に閲覧権限がないことを表現したい場合、```403```ステータスを使用する。ステータスコードについては、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_application_collaboration_api_restful.html

<br>

## 02-02. SSOの種類

### OAuth認証

#### ▼ OAuth認証とは

認証/認可フェーズ全体の中で、認可フェーズにOAuthプロトコルを使用したクライアントの照合方法を『OAuth認証』と呼ぶ。認証フェーズと認可フェーズでは、```3```個の役割が定義されていることを説明したが、OAuthプロトコル```2.0```では、より具体的に4つの役割が定義されている。OAuth認証には、仕組み別に『認可コードフロー』『インプリシットフロー』『リソースオーナー・パスワード・クレデンシャルズフロー』などのいくつかの種類がある。

| 具体例             | 説明                                                         | 補足                                                         |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| クライアントアプリ | リソースオーナに対するアクション機能を持つ。                 | OAuthの文脈では、ブラウザがクライアントと呼ばれないことに注意する。また、クライアントアプリとリソース間のデータ通信は、ブラウザを介したリダイレクトによって実現することに注意する。 |
| リソースオーナー   | クライアントを使用しているユーザー。                         |                                                              |
| 認可サーバー       | リソースサーバーがリソースオーナーにアクセスできるトークンを作成する。 | 認可サーバーがリダイレクト先のクライアントアプリケーションのURLをレスポンスに割り当てられるように、クライアントアプリケーションの開発者がURLを事前登録しておく必要がある。認可サーバーを利用する開発者用に、コンソール画面が用意されていることが多い。<br>ℹ️ 参考：https://qiita.com/TakahikoKawasaki/items/8567c80528da43c7e844 |
| リソースサーバー   | クライアントのアカウント情報を持っている。                   |                                                              |

#### ▼ 認可コードフローの場合

最も基本的な認可コードフローを説明する。

ℹ️ 参考：

- https://kb.authlete.com/ja/s/oauth-and-openid-connect/a/how-to-choose-the-appropriate-oauth-2-flow
- https://qiita.com/TakahikoKawasaki/items/200951e5b5929f840a1f
- https://boxil.jp/mag/a3207/

![Oauthの具体例](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/Oauthの具体例.png)

（１）ユーザーが、Facebookアカウントを使用してInstagramにログインしようとする。この時、ブラウザはFacebookにリクエストを送信する。

（２）Facebookはブラウザににアカウント連携の承認ボタンをレスポンスとして返信する。ユーザーは、表示された承認ボタンを押し、ブラウザはFacebookにリクエストを送信する。

（３）一時的に有効な認可コードを発行してもらうため、FacebookはInstagram認可サーバーに認可リクエストを送信する。

```http
GET https://www.instagram.com/auth?<下表で説明>
HOST: authorization-server.com # 認可サーバーのホスト
```

| クエリストリングの種類 | 値              | 必須/任意     |
| ---------------------- | --------------- | -------------- |
| response_type          | code            | 必須           |
| client_id              | クライアントID  | 必須           |
| redirect_uri           | リダイレクトURL | 条件により必須 |
| state                  | 任意の文字列    | 推奨           |
| scope                  | 認可スコープ    | 任意           |
| code_challenge         | チャレンジ      | 任意           |
| code_challege_method   | メソッド        | 任意           |

（４）Instagramの認可サーバーは認可リクエストを受信し、認可コードを発行する。また、Facebookにリダイレクトできるように、```Location```ヘッダーにURLと認可レスポンスパラメーターを割り当て、ブラウザにレスポンスを返信する。ブラウザはFacebookにリクエストを再送信し、Facebookは認可レスポンスパラメーターを受け取る。

```http
302 Found
Location: https://www.facebook.com/login?<下表で説明>
```

| クエリストリングのキーの種類 | 値           | 必須/任意                                                   |
| ---------------------------- | ------------ | ------------------------------------------------------------ |
| code                         | 認可コード   | 必須                                                         |
| state                        | 任意の文字列 | 認可リクエストのクエリストリングで、stateキーが使用されていれば必須 |

（５）Facebookは、認可コードを割り当てた認可リクエストをInstagramのサーバーに送信する。

```http
POST https://www.instagram.com/auth?
Host: authorization-server.com # 認可サーバーのホスト
Content-Type: application/x-www-form-urlencoded

# ボディ
# 下表で説明
```

| ボディのキーの種類 | 値                 | 必須/任意                                                   |
| ------------------ | ------------------ | ------------------------------------------------------------ |
| grant_type         | authorization_code | 必須                                                         |
| code               | 認可コード         | 必須                                                         |
| redirect_uri       | リダイレクトURL    | 認可リクエストのクエリストリングで、redirect_uriキーが使用されていれば必須 |
| code_verifier      | ベリファイア       | 認可リクエストのクエリストリングで、code_verifierキーが使用されていれば必須 |
| client_id          | クライアントID     | 条件により必須                                               |
| client_secret      | シークレット値     | 条件により必須                                               |

（６）Instagramは認可コードを照合し、アクセストークンを発行する。また、Facebookにリダイレクトできるように、```Location```ヘッダーにURLを割り当て、ブラウザにレスポンスを返信する。ブラウザからFacebookにレスポンスがリダイレクトされる。ブラウザはFacebookにリクエストを再送信する。

ℹ️ 参考：

- https://boxil.jp/mag/a3207/
- https://qiita.com/TakahikoKawasaki/items/8567c80528da43c7e844

```http
302 Found
Location: https://www.facebook.com/login
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"<アクセストークン>",       # 必須
  "token_type":"<トークンタイプ>",          # 必須
  "expires_in":<有効秒数>,                 # 任意
  "refresh_token":"<リフレッシュトークン>", # 任意
  "scope":"<認可スコープ>"                  # 要求したスコープ群と差異があれば必須
}
```

#### ▼ 使用される認証スキーム

OAuth認証では、認証スキーマとしてBearer認証が選択されることが多く、AWSやGitHubは、独自の認証スキームを使用している。なお、認可サーバーによって発行されたBearerトークンは、```Authorization```ヘッダー、リクエストボディ、クエリパラメータのいずれかに割り当てて送信できる。

#### ▼ 付与タイプ

認可サーバーによるOAuth認証のトークンの付与方法には種類がある。

ℹ️ 参考：https://oauth.net/2/grant-types/

| 付与タイプ名             | 説明                                                         | 使用例                                                       |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Authorization Code Grant | アプリケーションが他のAPIにアクセスする場合に使用する。推奨されている。<br>ℹ️ 参考：https://oauth.net/2/grant-types/authorization-code/ | 他のSNSアプリとのアカウント連携                              |
| Client Credentials Grant | 推奨されている。<br>ℹ️ 参考：https://oauth.net/2/grant-types/client-credentials/ |                                                              |
| Device Code              | 推奨されている。<br>ℹ️ 参考：https://oauth.net/2/grant-types/device-code/ |                                                              |
| Implicit Grant           | 非推奨されている。<br>ℹ️ 参考：https://oauth.net/2/grant-types/implicit/ |                                                              |
| Password Grant           | ユーザー名とパスワードを照合し、トークンを付与する。非推奨されている。<br>ℹ️ 参考：<br>・https://oauth.net/2/grant-types/password/<br>・https://developer.okta.com/blog/2018/06/29/what-is-the-oauth2-password-grant#the-oauth-20-password-grant | LaravelのPassword Grant Token機能は、Password Grantタイプを使用している。<br>ℹ️ 参考：https://readouble.com/laravel/8.x/ja/passport.html#password-grant-tokens |

<br>

### SAML認証：Security Assertion Markup Language

#### ▼ SAML認証とは

調査中...

<br>

### OpenID Connect

#### ▼ OpenID Connectとは

調査中...

#### ▼ 使用される認証スキーム

調査中...

<br>

## 03. JWT：JSON Web Token

### JWTとは

『ヘッダー』『ペイロード』『署名』のそれぞれのJSONデータをbase64方式によってエンコードし、ドットでつないだトークン。Bear認証やOauth認証のトークンとして使用できる。ランダムな文字列をこれら認証のトークンとするより、JWTを使用した方がより安全である。

ℹ️ 参考：

- https://meetup-jp.toast.com/3511
- https://dev.classmethod.jp/articles/json-signing-jws-jwt-usecase/

<br>

### 認証での利用

#### ▼ Bear認証の場合

```http
GET https://example.com/bar.php
authorization: Bearer <ヘッダーJSONエンコード値>.<ペイロードJSONエンコード値>.<署名JSONエンコード値>
```

<br>

### JWTの作成

#### ▼ JWT作成の全体像

JWTは以下のサイトから取得できる。

ℹ️ 参考：https://jwt.io/

JWTの作成時に、例えばJavaScriptであれば、以下のような処理が実行されている。

```javascript
// <ヘッダーエンコード値>.<ペイロードエンコード値>.<署名エンコード値>
const token = base64urlEncoding(header) + "." +
      base64urlEncoding(payload) + "." +
      base64urlEncoding(signature)
```

#### ▼ ヘッダーのJSONデータの作成

ヘッダーは以下のJSONデータで定義される。署名のための暗号化アルゴリズムは、『```HS256```』『```RS256```』『```ES256```』『```none```』から選択できる。

```javascript
const header = {
    "typ" : "JWT"    // JWTの使用
    "alg" : "HS256", // 署名のための暗号化アルゴリズム
}
```

#### ▼ ペイロードのJSONデータの作成

ペイロードは以下のJSONデータで定義される。ペイロードには、実際に送信したいJSONを設定する。必ず設定しなければならない『予約済みクレーム』と、ユーザー側が自由に定義できる『プライベートクレーム』がある。

| 予約済みクレーム名         | 役割                      | 例       |
| -------------------------- | ------------------------- | -------- |
| ```sub```：Subject         | 一意な識別子を設定する。  | ユーザーID |
| ```iss```：Issuer          |                           |          |
| ```aud```：Audience        |                           |          |
| ```exp```：Expiration Time | JWTの有効期限を設定する。 |          |
| ```jti```：JWT ID          |                           |          |

```javascript
const payload = {
    "sub": "foo",
    "aud": "foo",
    "iss": "https://example.com",
    "exp": 1452565628,
    "iat": 1452565568
}
```

#### ▼ 署名のJSONデータの作成

例えばJavaScriptであれば、以下のような処理が実行されている。

```javascript
const signature = HMACSHA256(
    base64urlEncoding(header) + "." + base64urlEncoding(payload),
    secret
)
```

<br>

### JWTのクライアント保持

#### ▼ 保持方法と安全度の比較

ℹ️ 参考：https://qiita.com/Hiro-mi/items/18e00060a0f8654f49d6#%E6%97%A9%E8%A6%8B%E8%A1%A8

| クライアント保持方法 | 組み合わせ             | おすすめ度 | コメント                                                     |
| :------------------- | ---------------------- | :--------- | :----------------------------------------------------------- |
| localStorage         |                        | 🔺〜✖️       | XSSでJWTが盗まれる可能性がある。                             |
| ```Cookie```ヘッダー | プリフライトリクエスト | 🔺          | Access-Control-Max-Ageの期間内だとCSRFでJWTが盗まれる可能性がある。 |
| ```Cookie```ヘッダー | CSRFトークン           | ⭕          |                                                              |
| SameSiteCookie       |                        | ⭕          | SPAとAPIが同一オリジンの必要がある。                         |

<br>
