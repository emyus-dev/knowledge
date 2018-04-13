# Firebase CLI [![Build Status](https://travis-ci.org/firebase/firebase-tools.svg?branch=master)](https://travis-ci.org/firebase/firebase-tools) [![Coverage Status](https://img.shields.io/coveralls/firebase/firebase-tools.svg?branch=master&style=flat)](https://coveralls.io/r/firebase/firebase-tools) [![Node Version](https://img.shields.io/node/v/firebase-tools.svg)](https://www.npmjs.com/package/firebase-tools) [![NPM version](https://badge.fury.io/js/firebase-tools.svg)](http://badge.fury.io/js/firebase-tools)

これらは Firebase コマンドラインインタフェース（CLI）ツールです。
それらは次の目的に使用できます。

* Firebase プロジェクトにコードと資産を配備する
* Firebase Hostingサイト用のローカルWebサーバを実行する
* Firebase データベースのデータと相互作用する
* Firebase Auth へのユーザのインポート/エクスポート

Firebase CLI を使い始めるには、以下のコマンドの全リストを読んだり、[ホスティングCLIの詳しいドキュメント](https://firebase.google.com/docs/hosting/quickstart?authuser=0)を参照してください。

## Installation - インストール方法

Firebase CLIをインストールするには、まず [Firebaseアカウントにサインアップ](https://firebase.google.com/?hl=ja) する必要があります

次に、[Node.js](https://nodejs.org/ja/) と [npm](https://npmjs.org/) をインストールする必要があります。 Node.jsをインストールすると、npmもインストールされるはずです。

npm がインストールされたら、次のコマンドを実行して Firebase CLI を入手します。

```bash
npm install -g firebase-tools
```

これにより、グローバルにアクセス可能な `firebase` コマンドが提供されます。

## Commands - コマンド

**`firebase --help` コマンドはは使用可能なコマンドを一覧表示し、firebase `<command> --help` は個々のコマンドの詳細を表示します。**

コマンドがプロジェクト固有のものである場合、アクティブなプロジェクトエイリアスを持つプロジェクトディレクトリの中か、 `-P <project_id>` フラグを付けた Firebase プロジェクトID を指定する必要があります。

利用可能なコマンドと、それらの機能の簡単な一覧を以下に示します。

### Administrative Commands - 管理コマンド

Command | Description
------- | -----------
**login** | Firebaseアカウントに認証します。 Webブラウザにアクセスする必要があります。
**logout** | Firebase CLI からサインアウトします。
**login:ci** | 非対話型環境で使用する認証トークンを生成します。
**list** | すべての Firebase プロジェクトのリストを出力します。
**setup:web** | Firebase JS SDK の SDK設定情報を出力します。
**use** | アクティブな Firebase プロジェクトを設定し、プロジェクトエイリアスを管理します。
**open** | ブラウザーを関連するプロジェクトリソースにすばやく開きます。
**init** | 現在のディレクトリに新しい Firebase プロジェクトを設定します。 このコマンドは、現在のディレクトリに `firebase.json` 設定ファイルを作成します。
**help** | CLIまたは特定のコマンドに関するヘルプ情報を表示します。

認証のためにローカルサーバを起動する代わりに `--no-localhost` を追加してログインします（つまり、`firebase login --no-localhost` ）。 あるインスタンスに SSH で接続し、そのマシン上で Firebase を認証する必要がある場合には、ユースケースがあります。

### Deployment and Local Development - デプロイとローカル開発

これらのコマンドを使用すると、Firebase ホスティングサイトをデプロイして対話できます。

Command | Description
------- | -----------
**deploy** | Firebaseプロジェクトをデプロイします。 `firebase.json` 設定とローカルプロジェクトフォルダに依存します。
**serve** | Firebase ホスティングの設定でローカル Webサーバを起動します。 `firebase.json` に依存します。

### Auth Commands - 認証コマンド

Command | Description
------- | -----------
**auth:import** | データファイルから Firebase にアカウントを一括インポートする。
**auth:export** | Firebase のアカウントを一括してデータファイルにエクスポートします。

詳細な文書は[こちら](https://firebase.google.com/docs/cli/auth?hl=ja)です。

### Database Commands - データベースコマンド

Command | Description
------- | -----------
**database:get** | 現在のプロジェクトのデータベースからデータを取得し、JSON として表示します。 インデックス付きデータのクエリをサポートします。
**database:set** | 現在のプロジェクトのデータベース内の指定された場所にあるすべてのデータを置き換えます。 ファイル、STDIN、またはコマンドライン引数から入力を受け取ります。
**database:push** | 新しいデータを、現在のプロジェクトのデータベース内の指定された場所にあるリストにプッシュします。 ファイル、STDIN、またはコマンドライン引数から入力を受け取ります。
**database:remove** | 現在のプロジェクトのデータベース内の指定された場所にあるすべてのデータを削除します。
**database:update** | 現在のプロジェクトのデータベース内の指定された場所で部分的な更新を実行します。 ファイル、STDIN、またはコマンドライン引数から入力を受け取ります。
**database:profile** | データベースの使用状況をプロファイルし、レポートを生成する。

### Cloud Firestore Commands - Cloud Firestore コマンド

Command | Description
------- | -----------
**firestore:delete** | 現在のプロジェクトのデータベースからドキュメントまたはコレクションを削除します。 サブコレクションの再帰的な削除をサポートします。
**firestore:indexes** | 現在のプロジェクトから展開されたすべてのインデックスを一覧表示します。

### Cloud Functions Commands - Cloud Functions コマンド

Command | Description
------- | -----------
**functions:log** | デプロイされた Cloud Functions からログを読み取ります。
**functions:config:set** | 現在のプロジェクトの Cloud Functions の実行時設定値を保存します。
**functions:config:get** | 現在のプロジェクトの Cluod Functions の既存の設定値を取得します。
**functions:config:unset** | 現在のプロジェクトの実行時設定から値を削除します。
**functions:config:clone** | あるプロジェクト環境から別のプロジェクト環境に実行時設定をコピーします。
**experimental:functions:shell** | 局所的に関数をエミュレートし、Node.js シェルを起動します。ここでは、これらのローカル関数をテストデータで呼び出すことができます。

### Hosting Commands - ホスティングコマンド

Command | Description
------- | -----------
**hosting:disable** | アクティブなプロジェクトの Firebase ホスティングトラフィックの提供を停止します。 このコマンドを実行すると、プロジェクトのホスティングURLに「サイトが見つかりません」というメッセージが表示されます。

## Using with CI Systems - CI システムでの使用

Firebase CLI は認証を完了するためにブラウザを必要としますが、CI やその他のヘッドレス環境と完全に互換性があります。

1. ブラウザを備えたマシン上で、Firebase CLI をインストールします。
2. `firebase login：ci` を実行してログインし、新しいアクセストークンを出力します（現在の CLI セッションは影響を受けません）。
3. 出力トークンは、安全ではあるがアクセス可能な方法で CIシステムに格納します。

Firebase コマンドの実行中にこのトークンを使用するには、次の2つの方法があります。

1. 環境変数 `FIREBASE_TOKEN` としてトークンを格納すると、自動的に利用されます。
2. CIシステムの `--token <token>` フラグですべてのコマンドを実行します。

トークンロードの優先順位は、フラグ、環境変数、アクティブなプロジェクトです。

Firebase CLIを使用しているマシンでは、`firebase logout --token <token>` を実行すると、指定されたトークンに対するアクセスを即座に取り消します。


## Using as a Module - モジュールの使用

Firebase CLIは、標準 Nodeモジュールとしてプログラムで使用することもできます。 これはあなたのマシンでのみ行うことができ、クラウド機能内では実行できません。 各コマンドは、オプションオブジェクトを取り、Promise を返す関数として公開されています。 例えば…

```js
var client = require('firebase-tools');
client.list().then(function(data) {
  console.log(data);
}).catch(function(err) {
  // handle error
});

client.deploy({
  project: 'myfirebase',
  token: process.env.FIREBASE_TOKEN,
  cwd: '/path/to/project/folder'
}).then(function() {
  console.log('Rules have been deployed!')
}).catch(function(err) {
  // handle error
});
```