<div align="center">

<h1>🎯 phishing-simulation</h1>

<h3>AWS で作る、標的型攻撃メールの訓練用ページ</h3>

<p>📦 <strong>Amazon S3</strong> ・ 🌐 <strong>Amazon CloudFront</strong> ・ 🗃️ <strong>Amazon DynamoDB</strong> ・ 📄 <strong>HTML</strong></p>

<p>AWS に詳しくない方でも作業できるように、<br>
押すボタンや入力内容を順番に説明します。</p>

</div>

---

> [!CAUTION]
> ⚠️ 必ず会社や組織の許可を得てから実施してください。パスワードや個人情報を入力させるページは作らないでください。

<p align="center">
  <a href="#overview">🗺️ しくみ</a> ・
  <a href="#words">📖 用語集</a> ・
  <a href="#steps">🚀 作り方</a> ・
  <a href="#dynamodb">🗃️ 追加設定</a> ・
  <a href="#check">✅ 最後の確認</a> ・
  <a href="#help">🆘 困ったとき</a>
</p>

<a id="overview"></a>

## 🗺️ 作るしくみ

```mermaid
flowchart LR
    A["👤 訓練を受ける人<br/>ブラウザー"]
    B["🌐 CloudFront<br/>Web サイトの入り口"]
    C["🔒 S3<br/>非公開のファイル置き場"]
    D["📄 index.html"]

    A -->|HTTPS でアクセス| B
    B -->|ファイルを読み込む| C
    C --> D

    classDef person fill:#e8f4ff,stroke:#1f6feb,color:#0d1117,stroke-width:2px;
    classDef cloud fill:#fff3cd,stroke:#f59e0b,color:#0d1117,stroke-width:2px;
    classDef storage fill:#e6ffed,stroke:#2da44e,color:#0d1117,stroke-width:2px;
    classDef file fill:#f3e8ff,stroke:#8250df,color:#0d1117,stroke-width:2px;

    class A person;
    class B cloud;
    class C storage;
    class D file;
```

- `S3` に Web ページのファイルを保存します。
- `CloudFront` を Web サイトの入り口にします。
- S3 は直接公開しません。CloudFront を通してページを表示します。

### 📍 作業の流れ

| 1️⃣ 保存場所を作る | 2️⃣ HTML を入れる | 3️⃣ Web 公開の入り口を作る | 4️⃣ トップページを決める | 5️⃣ 表示を確認する |
| :---: | :---: | :---: | :---: | :---: |
| 📦 S3 | 📄 `index.html` | 🌐 CloudFront | 🏠 Default root object | 🎉 完成 |

---

<a id="words"></a>

## 📖 よく出てくる言葉

| 言葉 | かんたんな意味 |
| --- | --- |
| S3 | ファイルを保存する場所 |
| バケット | S3 の中に作る、ファイル入れ |
| CloudFront | S3 のファイルを Web サイトとして表示するサービス |
| ディストリビューション | CloudFront で作る Web サイトの設定 |
| オリジン | CloudFront がファイルを取りに行く場所。この手順では S3 |
| デプロイ | 設定を AWS に反映すること |
| DynamoDB | トークンと氏名、トークンとアクセス日時などを保存するデータベース |
| テーブル | DynamoDB のデータをまとめて保存する場所 |
| パーティションキー | DynamoDB で項目を識別し、保存場所を決めるキー |
| ソートキー | 同じパーティションキーを持つ項目を並べ分けるキー |
| キャッシュ削除 | CloudFront に残っている古いファイルを無効にする操作 |

---

## ✅ 作業を始める前に

次の3点を確認してください。

- AWS マネジメントコンソールにサインインできる
- S3、CloudFront、DynamoDB を作成できる権限がある
- 訓練の対象者、実施日、問い合わせ先が決まっている

> [!NOTE]
> 💡 AWS の画面は変更されることがあります。ボタンの場所が少し違っても、同じ名前の項目を探してください。

---

<a id="steps"></a>

## 🚀 作り方

### 1️⃣ S3 にバケットを作る

> **進み具合:** 🟩 ⬜ ⬜ ⬜ ⬜

#### 1. S3 の画面を開く

1. AWS マネジメントコンソールにサインインします。
2. 画面上部の検索欄に `S3` と入力します。
3. 検索結果の「S3」をクリックします。
4. 左メニューの「バケット」→「汎用バケット」の順にクリックします。
5. 「バケットを作成」をクリックします。

![S3 の「バケットを作成」ボタン](docs/images/s3-create-bucket.png)

#### 2. バケットの設定を入力する

画面の項目を、上から順に設定します。

| 画面の項目 | 選ぶもの・入力するもの | 説明 |
| --- | --- | --- |
| AWS リージョン | 東京 `ap-northeast-1` | 東京になっていればそのまま |
| バケットタイプ | 汎用 | この手順で使うタイプ |
| バケット名前空間 | グローバル名前空間 | 変更しない |
| バケット名 | `<training-site-random-suffix>` | 自分で決めた名前に置き換える |
| オブジェクト所有者 | ACL 無効（推奨） | そのまま |
| パブリックアクセス | すべてブロックを ON | 必ず ON にする |
| バージョニング | 無効 | そのまま |
| タグ | なし | 追加しない |
| 暗号化 | SSE-S3 | そのまま |
| バケットキー | 初期設定のまま | 変更しない |
| 詳細設定 | 初期設定のまま | 変更しない |

> [!IMPORTANT]
> 🛠️ バケット名は、AWS 全体でほかの人と同じ名前を使えません。「この名前は使えません」と表示されたら、名前の後ろにランダムな英数字を追加してください。

#### 3. バケットを作る

1. 「パブリックアクセスをすべてブロック」が ON になっていることを、もう一度確認します。
2. 画面下部の「バケットを作成」をクリックします。
3. バケット一覧に、作ったバケットが表示されたら完了です。

---

### 2️⃣ `index.html` を S3 に入れる

> **進み具合:** 🟩 🟩 ⬜ ⬜ ⬜

#### 1. `index.html` を用意する

このプロジェクトフォルダーにある [`index.html`](./index.html) を使います。中身は次のとおりです。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Training Site</title>
</head>
<body>
  <h1>セキュリティ訓練ページ</h1>
  <p>このページは標的型訓練用です。</p>

  <p id="token-display"></p>

  <script>
    const params = new URLSearchParams(window.location.search);
    const token = params.get('token');

    document.getElementById('token-display').textContent =
      token ? `token = ${token}` : 'token が指定されていません';
  </script>
</body>
</html>
```

> [!IMPORTANT]
> 📄 ファイル名は `index.html` です。`Index.html` や `index.htm` に変えないでください。

#### 2. S3 にアップロードする

1. S3 のバケット一覧を開きます。
2. 先ほど作ったバケット名をクリックします。
3. 「オブジェクト」タブの「アップロード」をクリックします。
4. 「ファイルを追加」をクリックします。
5. パソコンに保存されている `index.html` を選びます。
6. 画面に `index.html` が表示されたら、右下の「アップロード」をクリックします。
7. 緑色の成功メッセージが表示されたら完了です。
8. バケットの中に `index.html` が表示されていることを確認します。

> [!NOTE]
> 💡 この時点で S3 の URL を開けなくても正常です。次の手順で CloudFront を設定します。

---

### 3️⃣ CloudFront を作る

> **進み具合:** 🟩 🟩 🟩 ⬜ ⬜

#### 1. CloudFront の画面を開く

1. AWS 画面上部の検索欄に `CloudFront` と入力します。
2. 検索結果の「CloudFront」をクリックします。
3. 定額プランの案内が表示されたら、「定額ディストリビューションを作成」をクリックします。

![CloudFront の「定額ディストリビューションを作成」ボタン](docs/images/cloudfront-flat-rate-plan.png)

4. 「Flat-rate plans」を選びます。
5. 無料プランの「Choose 無料」をクリックします。

> [!WARNING]
> 💰 画面に料金が表示されたら、作成する前に必ず確認してください。画面が画像と違う場合は、自分の画面に表示される案内に従ってください。

#### 2. 基本設定を入力する

「Get started」画面で設定します。

| 画面の項目 | 入力・選択するもの |
| --- | --- |
| Distribution name | 好きな名前。例: `<training-distribution>` |
| Description | 何も入力しない |
| Distribution type | Single website configuration |
| Route 53 managed domain | 何も入力しない |

入力できたら、「Next」をクリックします。

#### 3. S3 を選ぶ

「Specify origin」画面で設定します。

| 画面の項目 | 入力・選択するもの |
| --- | --- |
| Origin type | Amazon S3 |
| S3 origin | 「Browse S3」から、先ほど作ったバケットを選ぶ |
| Origin path | 何も入力しない |
| Allow private S3 bucket access to CloudFront | ON のまま |
| Origin settings | Use recommended origin settings |
| Cache settings | Use recommended cache settings tailored to serving S3 content |

S3 バケットの選び方は次のとおりです。

1. `S3 origin` の「Browse S3」をクリックします。
2. 先ほど作ったバケットを選びます。
3. 「Choose」をクリックします。
4. `S3 origin` に選んだバケットが表示されたことを確認します。
5. 「Next」をクリックします。

> [!IMPORTANT]
> 🔒 `Allow private S3 bucket access to CloudFront` は ON のままにしてください。S3 を直接公開せず、CloudFront からだけファイルを読めるようにするためです。

#### 4. セキュリティ画面を確認する

1. 「Enable security」画面が表示されます。
2. 有料機能の表示がある場合は、料金を確認します。
3. この手順では設定を変えず、「Next」をクリックします。

#### 5. CloudFront を作成する

1. 「Review and create」画面で、これまでの設定が表示されます。
2. `S3 origin` に正しいバケットが表示されていることを確認します。
3. 画面右下の「Create distribution」をクリックします。
4. 緑色の「新しいディストリビューションが正常に作成されました。」が表示されることを確認します。
5. 「最終変更日」が「デプロイ」になるまで待ちます。

> [!NOTE]
> ⏳ 設定の反映には時間がかかることがあります。すぐに完了しない場合は、少し待ってから画面を再読み込みしてください。

---

### 4️⃣ トップページに `index.html` を設定する

> **進み具合:** 🟩 🟩 🟩 🟩 ⬜

1. 作成した CloudFront ディストリビューションの詳細画面を開きます。
2. 「一般」タブをクリックします。
3. 「設定」の右上にある「編集」をクリックします。
4. `Default root object - optional` に `index.html` と入力します。

![Default root object に index.html を入力した画面](docs/images/cloudfront-default-root-object.png)

> [!IMPORTANT]
> ✍️ `index.html` の前に `/` は付けません。`/index.html` ではなく、`index.html` と入力してください。

5. 画面下部の「変更を保存」をクリックします。
6. 緑色の「ディストリビューション設定が正常に更新されました。」が表示されることを確認します。
7. 設定の反映が終わるまで待ちます。

---

### 5️⃣ Web ページを開く

> **進み具合:** 🟩 🟩 🟩 🟩 🟩

1. CloudFront の「一般」タブを開きます。
2. 「ディストリビューションドメイン名」を探します。
3. 表示されたドメイン名をコピーします。
4. ブラウザーのアドレス欄に、次のように入力します。

```text
https://<コピーしたドメイン名>/
```

5. 「セキュリティ訓練ページ」と表示されたら成功です。🎉

---

<a id="dynamodb"></a>

## 🗃️ DynamoDB とクエリパラメーターの追加設定

ここからは、訓練用のトークンに対応する氏名と、アクセス日時を保存するためのテーブルを作ります。その後、URL の `token` を `index.html` に表示できることを確認します。

> [!IMPORTANT]
> 🔌 この手順の `index.html` は、URL の `token` を画面に表示するだけです。DynamoDB から氏名を取得したり、アクセス日時を書き込んだりはしません。実際に連携するには、別途 API や Lambda などのバックエンドが必要です。

### 6️⃣ トークンと氏名のテーブルを作る

> **追加設定の進み具合:** 🟩 ⬜ ⬜ ⬜

#### 1. DynamoDB の画面を開く

1. AWS 画面上部の検索欄に `DynamoDB` と入力します。
2. 検索結果の「DynamoDB」をクリックします。
3. 左メニューの「テーブル」をクリックします。
4. 「テーブルの作成」をクリックします。

![DynamoDB の「テーブルの作成」ボタン](docs/images/dynamodb-create-table.png)

#### 2. `training-users` のキーを設定する

「テーブルの詳細」を、次のとおり設定します。

| 画面の項目 | 入力・選択するもの |
| --- | --- |
| テーブル名 | `training-users` |
| パーティションキー | `token` |
| パーティションキーのタイプ | 文字列 |
| ソートキー | 使わない |

![training-users のパーティションキー設定](docs/images/dynamodb-training-users-keys.png)

#### 3. キャパシティーを設定する

「テーブル設定」より下を、次のとおり設定します。

| 画面の項目 | 入力・選択するもの |
| --- | --- |
| テーブル設定 | 設定をカスタマイズ |
| テーブルクラス | DynamoDB 標準 |
| キャパシティーモード | プロビジョンド |
| 読み込みキャパシティーの Auto Scaling | オフ |
| 読み込みのプロビジョンドキャパシティーユニット | `1` |
| 書き込みキャパシティーの Auto Scaling | オフ |
| 書き込みのプロビジョンドキャパシティーユニット | `1` |

![DynamoDB のテーブルクラスとキャパシティー設定](docs/images/dynamodb-capacity-settings.png)

> [!WARNING]
> 💰 DynamoDB は設定や利用量によって料金が発生します。「テーブルの作成」を押す前に、AWS 画面に表示される料金案内を確認してください。

#### 4. テーブルを作る

1. 画面下部の「テーブルの作成」をクリックします。
2. 「テーブルは正常に作成されました。」と表示されるまで待ちます。
3. 一覧の状態が「アクティブ」になったら、`training-users` をクリックします。
4. 「テーブルアイテムの探索」をクリックします。

![作成した training-users と「テーブルアイテムの探索」ボタン](docs/images/dynamodb-training-users-created.png)

#### 5. 確認用の項目を1件作る

1. 「項目を作成」をクリックします。
2. `token` の値に `test-001` と入力します。
3. 「新しい属性の追加」→「文字列」を選びます。
4. 属性名に `name`、値に `テスト太郎` と入力します。
5. 右下の「項目を作成」をクリックします。

![token と name を入力して項目を作成する画面](docs/images/dynamodb-create-item.png)

> [!NOTE]
> 🧪 `test-001` と `テスト太郎` は動作確認用の値です。本物の氏名やメールアドレスなどは、許可なく登録しないでください。

---

### 7️⃣ トークンとアクセス日時のテーブルを作る

> **追加設定の進み具合:** 🟩 🟩 ⬜ ⬜

#### 1. 2つ目のテーブルのキーを設定する

1. DynamoDB の「テーブル」画面に戻ります。
2. 「テーブルの作成」をクリックします。
3. 「テーブルの詳細」を、次のとおり設定します。

| 画面の項目 | 入力・選択するもの |
| --- | --- |
| テーブル名 | `training-accesses` |
| パーティションキー | `token` |
| パーティションキーのタイプ | 文字列 |
| ソートキー | `accessedAt` |
| ソートキーのタイプ | 文字列 |

![training-accesses のパーティションキーとソートキー設定](docs/images/dynamodb-training-accesses-keys.png)

#### 2. キャパシティーを設定して作成する

1. `training-users` と同じく、「設定をカスタマイズ」を選びます。
2. テーブルクラスを「DynamoDB 標準」にします。
3. キャパシティーモードを「プロビジョンド」にします。
4. 読み込みと書き込みの Auto Scaling を、どちらもオフにします。
5. 読み込みと書き込みのプロビジョンドキャパシティーユニットを、どちらも `1` にします。
6. 「テーブルの作成」をクリックします。
7. 成功メッセージが表示され、状態が「アクティブ」になったら完了です。

---

### 8️⃣ `index.html` を更新して S3 に上書きする

> **追加設定の進み具合:** 🟩 🟩 🟩 ⬜

#### 1. URL の `token` を受け取る

このプロジェクトの [`index.html`](./index.html) には、次の処理が入っています。

```html
<p id="token-display"></p>

<script>
  const params = new URLSearchParams(window.location.search);
  const token = params.get('token');

  document.getElementById('token-display').textContent =
    token ? `token = ${token}` : 'token が指定されていません';
</script>
```

- URL に `?token=test-001` がある場合は、`token = test-001` と表示します。
- `token` がない場合は、「token が指定されていません」と表示します。

> [!IMPORTANT]
> 🔒 URL に付けるトークンには、氏名、メールアドレス、社員番号などの個人情報を直接入れないでください。訓練専用のランダムな値を使ってください。

#### 2. S3 に上書きアップロードする

1. S3 の「汎用バケット」を開きます。
2. Web ページを保存しているバケット名をクリックします。
3. 「オブジェクト」タブの「アップロード」をクリックします。
4. 「ファイルを追加」から、更新した `index.html` を選びます。
5. 同名ファイルを上書きする内容になっていることを確認します。
6. 右下の「アップロード」をクリックします。
7. 緑色の成功メッセージが表示されたら完了です。

![S3 の「アップロード」ボタンと index.html](docs/images/s3-overwrite-index.png)

---

### 9️⃣ CloudFront のキャッシュを削除して確認する

> **追加設定の進み具合:** 🟩 🟩 🟩 🟩

#### 1. キャッシュ削除を開く

1. CloudFront の「ディストリビューション」を開きます。
2. Web ページに使っているディストリビューションをクリックします。
3. 「キャッシュ削除」タブをクリックします。
4. 「キャッシュ削除を作成」をクリックします。

![CloudFront の「キャッシュ削除を作成」ボタン](docs/images/cloudfront-create-invalidation.png)

#### 2. すべてのファイルを無効にする

1. `Selection method` は `By paths` のままにします。
2. `Object paths to invalidate` に `/*` と入力します。
3. 「キャッシュ削除を作成」をクリックします。
4. キャッシュ削除のステータスが完了になるまで待ちます。

![Object paths to invalidate に /* を入力した画面](docs/images/cloudfront-invalidation-path.png)

> [!NOTE]
> 💡 `/*` は、CloudFront に残っているすべてのパスのキャッシュを無効にする指定です。

#### 3. `token` の表示を確認する

ブラウザーで、次の URL を開きます。

```text
https://<ディストリビューションドメイン名>/?token=test-001
```

画面に `token = test-001` と表示されたら成功です。🎉

![クエリパラメーターの token が表示された画面](docs/images/cloudfront-token-result.png)

---

<a id="check"></a>

## ✅ 最後の確認

- [ ] S3 の「パブリックアクセスをすべてブロック」が ON
- [ ] S3 バケットの中に `index.html` がある
- [ ] CloudFront の `S3 origin` に正しいバケットが表示されている
- [ ] `Allow private S3 bucket access to CloudFront` が ON
- [ ] `Default root object` が `index.html`
- [ ] CloudFront の URL で Web ページを開ける
- [ ] URL が `https://` で始まっている
- [ ] DynamoDB に `training-users` テーブルがあり、状態が「アクティブ」
- [ ] `training-users` のパーティションキーが `token`（文字列）
- [ ] `training-users` に `token: test-001` と `name: テスト太郎` の確認用項目がある
- [ ] DynamoDB に `training-accesses` テーブルがあり、状態が「アクティブ」
- [ ] `training-accesses` のキーが `token`（文字列）と `accessedAt`（文字列）
- [ ] S3 の `index.html` を更新済み
- [ ] CloudFront の `/*` のキャッシュ削除が完了している
- [ ] `?token=test-001` を付けると `token = test-001` と表示される

---

<a id="help"></a>

## 🆘 うまくいかないとき

| 画面に出るもの | 確認すること |
| --- | --- |
| `403` エラー | `Default root object` が `index.html` か確認。`S3 origin` と S3 のプライベートアクセスも確認 |
| `404` エラー | S3 バケットの中に `index.html` があるか確認 |
| 古いページが表示される | 数分待ってからブラウザーを再読み込み |
| S3 の URL を開けない | 正常な動き。CloudFront の URL を開く |
| DynamoDB のテーブルを作成できない | テーブル名とキー名を確認。DynamoDB を作成できる権限があるか管理者に確認 |
| DynamoDB の状態が「作成中」のまま | 少し待ってから、更新ボタンで画面を再読み込み |
| `token が指定されていません` と表示される | URL の末尾が `/?token=test-001` になっているか確認 |
| 更新前の `index.html` が表示される | S3 の上書きアップロードと CloudFront の `/*` のキャッシュ削除を確認 |

---

## 🔒 安全に使うために

- 訓練の許可を得た人だけが使用してください。
- パスワード、氏名、メールアドレスを入力させる画面は作らないでください。
- URL の `token` には、個人情報や認証情報を直接入れないでください。
- DynamoDB に実データを登録する場合は、保存目的、閲覧権限、保存期間を組織内で決めてください。
- S3 のパブリックアクセスは、すべてブロックしてください。
- 訓練が終わったら、不要になった CloudFront、S3、DynamoDB のテーブルを削除してください。
