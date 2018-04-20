# Browserify-HMR

これは、Webpack の [Hot Module Replacement API](https://webpack.github.io/docs/hot-module-replacement.html) を Browserify のプラグインとして実装したものです。このプロジェクトは多くの場合動作すると思われますが、開発の初期段階にあり、現時点でいくつかのバグがある可能性があります。それがあなたのためにどのように機能するか教えてください！

## Quick Example

```bash
git clone https://github.com/AgentME/browserify-hmr.git
cd browserify-hmr/example
npm i && npm start
```

[http://localhost:8080/](http://localhost:8080/) を開き、 `label.jsx` と `interval.js` を更新してみてください。

## Hot Module Replacement Usage

ホットモジュールリプレースは、更新されたモジュールを再実行することによって機能します。
[Hot Module Replacement API](https://webpack.github.io/docs/hot-module-replacement.html) を使用して、どのモジュールが更新を受け入れることができるか、そしてモジュールが更新されるときに何をするかを定義する必要があります。

ただし、HMR API をアプリケーションコードで直接使用することは、常に最適なルートではありません。 [react-transform-hmr](https://github.com/gaearon/react-transform-hmr) や [ud](https://github.com/AgentME/ud) のようなコード変換やライブラリは、一般的な作業を行うのに役立ち、ある種のコードをホット・リプレース可能にすることを完全に自動化します。

Webpack ホットモジュールリプレース API の `module.hot.*` 機能に加えて、以下も実装されています。

### module.hot.setUpdateMode(mode, options)

これにより、実行時にバンドルの更新モードとオプションを変更することができます。 `mode` は文字列でなければならず、 プラグインオプションズセクションの `mode` と同じ意味を持ちます。 `options` は `url`、`cacheBust`、`ignoreUnaccepted` プロパティを持つオプションオブジェクトです。プラグインオプションズセクションの同じオプションと同じ意味です。これが呼び出されると、HMR ステータスは「アイドル」でなければなりません。

## Plugin Usage

watchify コールに browserify-hmr プラグインを追加します。

```bash
npm i browserify-hmr
watchify -p browserify-hmr index.js -o bundle.js
```

Browserify-HMR も Node と連携します！ m/mode オプションを指定して、 "fs" メソッドを使用して自身を更新するように指示します。 以下のオプションのセクションを参照してください。

```bash
watchify --node -p [ browserify-hmr -m fs ] index.js -o bundle.js
```

Watchify は必須ではありません。 リロードのタイミングをより詳細に制御したい場合は、Browserify を手動で複数回実行することができます。

```bash
browserify -p [ browserify-hmr -m ajax -u /bundle.js ] index.js -o bundle.js
nano foo.js # make some edits
nano bar.js # edit some more files
browserify -p [ browserify-hmr -m ajax -u /bundle.js ] index.js -o bundle.js
```

## Plugin Options

Browserify-HMR オプションは、プラグイン名の後のコマンドラインから長いか短い形式の中かっこで指定することができます。

```bash
watchify -p [ browserify-hmr -m fs ] index.js -o bundle.js
```

オプションは、Browserify API を使用して指定することもできます。

```javascript
var hmr = require("browserify-hmr");

browserify().plugin(hmr, {
  mode: "fs"
});
```

`m、mode` は、更新モードを設定する文字列です。
"websocket" は、バンドルにプラグインによってホストされている websocket サーバーへの接続を開いて、変更を待機するように指示します。 websocket は、tlskey、tlscert、または tlsoptions オプションのいずれかが渡されない限り、HTTP 経由で提供されます。
"ajax" は AJAX リクエストを使ってアップデートをダウンロードします。
"fs" はファイルシステムモジュールを使用し、Node の使用に適しています。
"none" を指定すると、バンドルは更新をチェックするように構成されません。 ランタイムに `module.hot.setUpdateMode` を呼び出してバンドルを再設定することができます。 デフォルトは "websocket" です。

```bash
watchify -p [ browserify-hmr -m none --supportModes [ ajax websocket ] ] index.js -o bundle.js
```

`noServe` は、Browserify-HMR の自動 WebSocket サーバを無効にするブール値です。通常、Browserify-HMR プラグインは、`mode` または `supportModes` に "websocket" が含まれている場合、websocket サーバーを自動的にホストします。これが `true` に設定されている場合、プラグインは自分の websocket サーバーをホストしません。これは、websocket モードでバンドルを構築していて、`url` オプションを Browserify-HMR の別のインスタンスによってホストされている websocket サーバーを指すように設定している場合に使用できます。 デフォルトは false です。

`ignoreUnaccepted` は、 "websocket" モードの場合、 `ignoreUnaccepted` パラメータの値を `module.hot.apply` に制御するブール値です。（ "websocket" モードが使用されている場合、Browserify-HMR は自動的に更新をチェックして適用するため、アプリケーションは `module.hot.apply` を呼び出すことはありません）。

`u、url` は、websocket 接続または Browserify バンドルにアクセスできる更新 URL を設定する文字列です。
"websocket" モードでは、これはデフォルトで `"http://localhost:3123"` になります。  
これは "ajax" モードに必要です。これは "fs" モードには必要ありません。

`p、port` は、 "websocket" モードが使用されている場合にポートをリッスンするように設定する番号です。  
これを変更すると、ほとんどの場合、`url` 設定も変更したいと思うでしょう。 デフォルトは `3123` です。

`h、hostname` は "websocket" モードが使用されている場合に listen するホスト名です。これはデフォルトで `"localhost"` になります。

`b、cacheBust` は、AJAX 要求に対してキャッシュ破棄を使用するかどうかを制御するブール値です。  
これは、更新モードが "ajax" に設定されている場合にのみ有効です。 `true` の場合、リクエストごとにランダムパラメータが URL に追加されます。 これにより、サーバが低い Expires または Cache-Control ヘッダ値を与えない場合に、キャッシュをバイパスすることができます。これにより、E-Tag ヘッダーと Last-Modified ヘッダーがクライアントによって使用されるのを防ぐので、必要でない場合は無効にしておくとパフォーマンスが向上する可能性があります。これを調整する前に、スクリプトが提供されている HTTP ヘッダーを調整することを検討する必要があります。デフォルトは `false` です。

`k、key` はバンドルキーです。 Browserify-HMR を使用して構築された複数のバンドルが同じ javascript 環境内で実行される場合は、それぞれに一意のバンドルキーが必要です。  
バンドルキーのデフォルト値は、更新モードと更新 URL を組み合わせて作成された値であるため、一般的にこのオプションについて心配する必要はありません。

`K、tlskey` は、HTTPS モードで使用するキーファイルへのパスです。

`C、tlscert` は、HTTPS モードで使用する証明書ファイルへのパスです。

`tlsoptions` は、 `https.createServer` の呼び出しに渡すオプションのオブジェクトです。  
このオブジェクトは JSONifiable でなければならないので、その内部のバッファの代わりに文字列を使用してください。  
このオプションは、コマンド行では指定できません。

デフォルトの websocket 更新モードを使用しない場合、browserify-hmr が更新を確認して適用する必要があるときに手動で通知する必要があります。  
プロジェクトのどこかで次のようなコードを使用して、更新をポーリングすることができます。

```javascript
if (module.hot) {
  var doCheck = function() {
    module.hot.check(function(err, outdated) {
      if (err) {
        console.error("Check error", err);
      }
      if (outdated) {
        module.hot.apply(function(err, updated) {
          if (err) {
            console.error("Update error", err);
          } else {
            console.log("Replaced modules", updated);
          }
          setTimeout(doCheck, 2000);
        });
      } else {
        setTimeout(doCheck, 2000);
      }
    });
  };
  doCheck();
}
```
