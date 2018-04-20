# watchify

[browserify](https://github.com/substack/node-browserify) ビルドの監視モード

ソースファイルを更新すると、browserify バンドルはその場で再コンパイルされます。

## example

```bash
$ watchify main.js -o static/bundle.js
```

ファイルを更新すると、`static/bundle.js` が自動的に段階的に再構築されます。

`-o` オプションは、パイプ入力を受け取るファイルまたはシェルコマンド（Windows では使用できません）です。

```bash
watchify main.js -o 'exorcist static/bundle.js.map > static/bundle.js' -d
```

```bash
watchify main.js -o 'uglifyjs -cm > static/bundle.min.js'
```

`-v` を使用すると、ファイルが書き込まれたときとバンドルに要した時間（秒単位）を表示し、より詳細な出力を得ることができます。

```bash
$ watchify browser.js -d -o static/bundle.js -v
610598 bytes written to static/bundle.js (0.23 seconds) at 8:31:25 PM
610606 bytes written to static/bundle.js (0.10 seconds) at 8:45:59 PM
610597 bytes written to static/bundle.js (0.14 seconds) at 8:46:02 PM
610606 bytes written to static/bundle.js (0.08 seconds) at 8:50:13 PM
610597 bytes written to static/bundle.js (0.08 seconds) at 8:58:16 PM
610597 bytes written to static/bundle.js (0.19 seconds) at 9:10:45 PM
```

## usage

`-o`（または `--outfile` ）が必須であることを除いて、browserify と同じオプションを指定して watchify を使用してください。 さらに、以下もあります。

```bash
標準 オプション:

  --outfile=FILE, -o FILE

    このオプションは必須です。 このファイルに browserify バンドルを書き込みます。 ファイルに `|` や  `>` の演算子が含まれている場合は、シェルコマンドとして扱われ、出力はパイプ演算されます。

  --verbose, -v                     [default: false]

    ファイルが書き込まれた時刻と、バンドルに要した時間（秒単位）を表示します。

  --version

    watchify と browserify のバージョンをモジュールパスとともに表示します。
```

```bash
高度なオプション:

  --delay                           [default: 100]

    変更後に「更新」イベントが発生するまでに待機する時間（ミリ秒単位）。

  --ignore-watch=GLOB, --iw GLOB    [default: false]

    パターンに一致する変更の監視ファイルを無視します。 パターンを省略すると、デフォルトで "**/node_modules/**" になります。

  --poll=INTERVAL                   [default: false]

    ポーリングを使用して変更を監視します。 間隔を省略すると、デフォルトで100msになります。 このオプションは、NFSボリュームを見ているときに便利です。
```

## methods

```javascript
var watchify = require("watchify");
```

### watchify(b, opts)

watchify は [browserify プラグイン](https://github.com/substack/node-browserify#bpluginplugin-opts)なので、他のプラグインと同様に適用できます。ただし、browserify インスタンス `b` を作成するときは、 `cache` および `packageCache` プロパティを設定する必要があります。

```javascript
var b = browserify({ cache: {}, packageCache: {} });
b.plugin(watchify);
```

```javascript
var b = browserify({
  cache: {},
  packageCache: {},
  plugin: [watchify]
});
```

**デフォルトでは、watchify は出力を表示しません。詳細については[イベント](https://github.com/substack/watchify#events)を参照してください。**

`b` は、ファイル内容をキャッシュし、ファイルが変更されたときに `'update'` イベントを発行する点を除いて browserify インスタンスのように動作し続けます。新しいバンドルを生成するために `'update'` イベントが発生した後で `b.bundle()` を呼び出す必要があります。最初に `b.bundle()` を余分に呼び出すと、キャッシングのために非常に高速になります。

**重要：** Watchify は、 `b.bundle()` を一度呼び出すと、それが返すストリームを完全にはけるまで、 `'update'` イベントを送出しません。

```javascript
var fs = require("fs");
var browserify = require("browserify");
var watchify = require("watchify");

var b = browserify({
  entries: ["path/to/entry.js"],
  cache: {},
  packageCache: {},
  plugin: [watchify]
});

b.on("update", bundle);
bundle();

function bundle() {
  b.bundle().pipe(fs.createWriteStream("output.js"));
}
```

### options

watchify の 2 番目のパラメータとして追加のオプションオブジェクトを渡すことができます。プロパティは次のとおりです。

`opts.delay` は、変更後に「更新」イベントを発行するまでに待機する時間（ミリ秒単位）です。 デフォルトは `100` です。

`opts.ignoreWatch` は、監視ファイルの変更を無視します。 `true` に設定すると、 `**/node_modules/**` は無視されます。 他の可能な値については、Chokidar の「無視された」[ドキュメント](https://github.com/paulmillr/chokidar#path-filtering)を参照してください。

### b.close()

開いているウォッチハンドルをすべて閉じます。

## events

### b.on('update', function (ids) {})

バンドルが変更されたら、変更されたバンドル `ids` の配列を送出します。

### b.on('bytes', function (bytes) {})

バンドルが生成されると、このイベントはバイト数で起動します。

### b.on('time', function (time) {})

バンドルが生成されると、このイベントはミリ秒単位でバンドルを作成するのにかかった時間で起動します。

### b.on('log', function (msg) {})

このイベントは、次の形式のメッセージでバンドルが作成された後に発生します。

```bash
X bytes written (Y seconds)
```

バンドル X のバイト数と秒数 Y の時間で計算されます。
