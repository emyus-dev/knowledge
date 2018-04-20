# browserify

ブラウザで `require('modules')` する。

[node](https://nodejs.org/ja/)-形式で `require()` を使用してブラウザコードを整理し、[npm](https://www.npmjs.com/)でインストールされたモジュールをロードします。

browserify は、ブラウザに提供できるバンドルを 1 つの `<script>` タグでビルドするために、アプリケーション内のすべての `require()` 呼び出しを再帰的に分析します。

![browserify](https://github.com/browserify/browserify/raw/master/assets/logo.png)

## getting started

初めてブラウジングする場合は、[browserify ハンドブック](./HANDBOOK.md)と [browserify.org](http://browserify.org/)のリソースを確認してください。

## example

いくつかの `require()` を含む `main.js` ファイルを拾い上げてください。
`'./foo.js'` や `'../lib/bar.js'` のような相対パスや、[ノードのモジュールルックアップアルゴリズム](https://github.com/browserify/resolve)を使って `node_modules/` を検索する `'gamma'` のようなモジュールパスを使用することができます。

```javascript
var foo = require("./foo.js");
var bar = require("../lib/bar.js");
var gamma = require("gamma");

var elem = document.getElementById("result");
var x = foo(100) + bar("baz");
elem.textContent = gamma(x);
```

`module.exports` または `exports` に割り当てることによって機能をエクスポートします。

```javascript
module.exports = function(n) {
  return n * 111;
};
```

では `browserify` コマンドを使用して `main.js` から始まるバンドルを構築しましょう。

```bash
$ browserify main.js > bundle.js
```

`main.js` が必要とするすべてのモジュールは、 [required](https://github.com/defunctzombie/node-required) を使用した `require()` グラフの再帰的ウォークから `bundle.js` に必要なもが含まれるようになっています。

このバンドルを使用するには、 `<script src = "bundle.js"></ script>` をあなたの html に追記して下さい！

## install

[npm](https://www.npmjs.com/) を使う

```bash
npm install -g browserify
```

## usage

```bash
Usage: browserify [entry files] {OPTIONS}

標準的なオプション:

    --outfile, -o  このファイルに browserify バンドルを書き込みます。
                   指定されていない場合、browserify は標準出力に出力します。

    --require, -r  bundle.require() へのモジュール名またはファイル
                   必要に応じて : セパレータを使用してターゲットを設定します。

      --entry, -e  アプリのエントリーポイント

     --ignore, -i  ファイルを空のスタブに置き換えます。 ファイル名にはワイルドカードが利用ができます。

    --exclude, -u  出力バンドルからファイルを省略します。 ファイル名にはワイルドカードが利用ができます。

   --external, -x  別のバンドルからファイルを参照する。 ファイル名にはワイルドカードが利用ができます。

  --transform, -t  トップレベルのファイルには変換モジュールを使用します。

    --command, -c  トップレベルのファイルでは、トランスフォームコマンドを使用します。

  --standalone -s  指定されたエクスポート名の UMD バンドルを生成します。
                   このバンドルは、他のモジュールシステムと連携し、モジュールシステムが見つからない場合は、指定された名前をウィンドウグローバルとして設定します。

       --debug -d  ファイルを個別にデバッグできるソースマップを有効にします。

       --help, -h  このメッセージを表示します。

  高度なオプションのヘルプは、 `browserify --help advanced` を実行する。

Specify a parameter.
```

```bash
高度なオプション:

  --insert-globals, --ig, --fast    [default: false]

    検出をスキップし、常に process、global、__filename、および __dirname の定義を挿入します。

    効果: ビルドが早くなる
    コスト: 少々

  --insert-global-vars, --igv

    検出および定義するグローバル変数のカンマ区切りリスト。
    デフォルト: __filename,__dirname,process,Buffer,global

  --detect-globals, --dg            [default: true]

    process、global、__filename、および __dirname の存在を検出し、存在する場合はこれらの値を定義します。

    効果: npm モジュールが動作する可能性が上がる
    コスト: ビルドが遅くなる

  --ignore-missing, --im            [default: false]

    何にも解決されない  `require（）` ステートメントは無視します。

  --noparse=FILE

    FILE をまったく解析させない。 これは、jquery や threejs のような巨大なライブラリの場合にはるかに高速です。

  --no-builtins

    組み込み関数をオフにします。 これは、コアの組み込み関数を提供するノードでバンドルを実行したいときに便利です。

  --no-commondir

    共用の設定をオフにします。 これは、バンドルが生成された元のパスを保持する場合に便利です。

  --no-bundle-external

    すべての外部モジュールのバンドルをオフにします。 ローカルファイルをバンドルしたい場合に便利です。

  --bare

    --no-builtins、--no-commondir の両方のエイリアス、--insert-global-vars を "__filename、__ dirname" だけに設定します。 これはノード内でバンドルを実行したい場合に便利です。

  --no-browser-field, --no-bf

    package.json browser のフィールド解決をオフにします。 これは、ノードでバンドルを実行する必要がある場合にも便利です。

  --transform-key

    デフォルトの package.json＃browserify＃transform フィールドの代わりに、browserify の実行時に適用されるすべての変換をリストする変換フィールド、カスタムフィールド（例： package.json＃browserify＃production または package.json＃browserify＃staging は、たとえば以下を実行することによって使用できます。
    * `browserify index.js --transform-key=production > bundle.js`
    * `browserify index.js --transform-key=staging > bundle.js`

  --node

    --bare と --no-browser-field のエイリアス。

  --full-paths

    モジュールIDの数値インデックスへの変換をオフにします。 これは、バンドルが生成された元のパスを保持するのに便利です。

  --deps

    標準バンドル出力の代わりに、module-deps によって生成された依存関係配列を出力します。

  --no-dedupe

    除外をオフにします。

  --list

    依存関係グラフの各ファイルを印刷します。 makefiles に役立ちます。

  --extension=EXTENSION

    モジュールとして EXTENSION を指定したファイルを考えてください。このオプションは複数回使用できます。

  --global-transform=MODULE, -g MODULE

    通常の変換が実行された後は、すべてのファイルに対して変換モジュールを使用します。

  --ignore-transform=MODULE, -it MODULE

    他の場所で指定されていても、特定の変換を実行しないでください。

  --plugin=MODULE, -p MODULE

    MODULE をプラグインとして登録します。

変換とプラグインへの引数の受け渡し：

  -t、-g、および -p の場合、subarg 構文を使用して、トランスフォームまたはプラグイン関数にオプションを2番目のパラメータとして渡すことができます。 例えば…

    -t [ foo -x 3 --beep ]

  次の関数を呼び出すことで、該当するファイルごとに `foo` トランスフォームを呼び出します。

    foo(file, { x: 3, beep: true })
```

## compatibility

IO をしない多くの [npm](https://www.npmjs.com/) モジュールは、ブラウジング後に動作します。 その他に動作するものがあります。

多くの node 組み込みモジュールは、明示的に `require()` またはその機能を使用する場合にのみ、ブラウザで動作するようにラップされています。

これらのモジュールのいずれかを `require()` すると、ブラウザ固有のシムが得られます。

* [assert](https://www.npmjs.com/package/assert)
* [buffer](https://www.npmjs.com/package/buffer)
* [console](https://www.npmjs.com/package/console-browserify)
* [constants](https://www.npmjs.com/package/constants-browserify)
* [crypto](https://www.npmjs.com/package/crypto-browserify)
* [domain](https://www.npmjs.com/package/domain-browser)
* [events](https://www.npmjs.com/package/events)
* [http](https://www.npmjs.com/package/stream-http)
* [https](https://www.npmjs.com/package/https-browserify)
* [os](https://www.npmjs.com/package/os-browserify)
* [path](https://www.npmjs.com/package/path-browserify)
* [punycode](https://www.npmjs.com/package/punycode)
* [querystring](https://www.npmjs.com/package/querystring-es3)
* [stream](https://www.npmjs.com/package/stream-browserify)
* [string_decoder](https://www.npmjs.com/package/string_decoder)
* [timers](https://www.npmjs.com/package/timers-browserify)
* [tty](https://www.npmjs.com/package/tty-browserify)
* [url](https://www.npmjs.com/package/url)
* [util](https://www.npmjs.com/package/util)
* [vm](https://www.npmjs.com/package/vm-browserify)
* [zlib](https://www.npmjs.com/package/browserify-zlib)

さらに、これらの変数を使用すると、それらはバンドルされた出力でブラウザに適した方法で定義されます。

* [process](https://www.npmjs.com/package/process)
* [Buffer](https://www.npmjs.com/package/buffer)
* global - トップレベルスコープの object (window)
* \_\_filename - 現在実行中のファイルのファイルパス
* \_\_dirname - 現在実行中のファイルのディレクトリパス

## more examples

### external requires

`require()` 関数をエクスポートするバンドルを簡単に作成するだけで、別のスクリプトタグからモジュールを`require()` できるようになります。ここでは、[スルー](https://www.npmjs.com/package/through)モジュールと[デュプレクサ](https://www.npmjs.com/package/duplexer)モジュールで `bundle.js` を作成します。

```bash
$ browserify -r through -r duplexer -r ./my-file.js:my-module > bundle.js
```

以下をページに埋め込むことで実行できるようになります。

```html
<script src="bundle.js"></script>
<script>
  var through = require('through');
  var duplexer = require('duplexer');
  var myModule = require('my-module');
  /* ... */
</script>
```

### external source maps

ソースマップを別の `.js.map` ソースマップファイルに保存したい場合は、それを達成するためにエクソシストを使用することができます。 それは次のように簡単です。

```bash
$ browserify main.js --debug | exorcist bundle.js.map > bundle.js
```

その他のオプションについては[こちら](https://github.com/thlorenz/exorcist#usage)をご覧ください。

### multiple bundles

browserify がすでにページスコープ内に定義されている `required` 関数を見つけた場合、バンドルされたモジュールのセットで一致するものが見つからなかった場合、その関数にフォールバックします。

このように、browserify を使用すると、バンドルを複数のページに分割して、まれに共有される共有モジュールのキャッシュの利点を得ることができますが、依然として `require()` を使用できます。共通の依存関係を除外するために `--external` と `--require` を組み合わせて使用してください。

たとえば、2 ページ、 `beep.js` のウェブサイトの場合…

```javascript
var robot = require("./robot.js");
console.log(robot("beep"));
```

次に `boop.js`

```javascript
var robot = require("./robot.js");
console.log(robot("boop"));
```

どちらも `robot.js` に依存します。

```javascript
module.exports = function(s) {
  return s.toUpperCase() + "!";
};
```

```bash
$ browserify -r ./robot.js > static/common.js
$ browserify -x ./robot.js beep.js > static/beep.js
$ browserify -x ./robot.js boop.js > static/boop.js
```

その後、ビープ音ページに以下のように利用できます。

```html
<script src="common.js"></script>
<script src="beep.js"></script>
```

ブープ音ページでは以下のように利用できます。

```html
<script src="common.js"></script>
<script src="boop.js"></script>
```

`-r` と `-x` を使ったこのアプローチは、少数の分割資産に対してうまく機能しますが、コンポーネントを自動的にファクタリングするためのプラグインがあります。これらのプラグインは、[browserify ハンドブックのパーティション分割のセクション](./HANDBOK.md)で説明しています。

### api example

API も直接使用できます。

```javascript
var browserify = require("browserify");
var b = browserify();
b.add("./browser/main.js");
b.bundle().pipe(process.stdout);
```

## methods

```javascript
var browserify = require("browserify");
```

### `browserify([files] [, opts])`

新しい browserify インスタンスを返します。

#### files

エントリファイルを指定する文字列、ファイルオブジェクト、またはそれらの型の配列（それらは混在している可能性があります）。

#### opts

オブジェクト。

`files` と `opts` は両方ともオプションですが、両方が渡された場合は、示された順序でなければなりません。

エントリーファイルは、 `files` と `/`、 または `opts.entries` で渡すことができます。

外部要求は、`files` 引数と同じ形式を受け入れ、 `opts.require` で指定することができます。

エントリファイルがストリームの場合、その内容が使用されます。ストリーミングファイルを使用する場合は、 `opts.basedir` を渡して、相対的な要求を解決できるようにしてください。

`opts.entries` は `files` と同じ定義を持っています。

`opts.noParse` は、配列内の各ファイルのすべての `require()` およびグローバル解析をスキップする配列です。
これは jquery や threejs のような巨大なライブラリには必要ないし、ノードスタイルのグローバルはありませんが、永遠に解析することができます。

`opts.transform` は、変換関数またはモジュール名の配列で、解析前にソースコードを変換します。

`opts.ignoreTransform` は他の場所で指定されていても実行されない変換の配列です。

`opts.plugin` は、使用するプラグイン関数またはモジュール名の配列です。
詳細については、以下のプラグインのセクションを参照してください。

`opts.extensions` は、拡張モジュールが指定されていないときに使用するモジュール参照機構用のオプションの追加拡張モジュールの配列です。
デフォルトでは、browserify はそのような場合には `.js` ファイルと `.json` ファイルのみを考慮します。

`opts.basedir` は、browserifyが `.` で始まるファイル名のバンドルを開始するディレクトリです。

`opts.paths` は、相対パスを使用して参照されていないモジュールを探すときにブラウジング検索を行うディレクトリの配列です。 `basedir` に絶対的でも相対的でもかまいません。 browserify コマンドを呼び出すときに `NODE_PATH` 環境変数を設定するのと同じです。

`opts.commondir` は、共通パスを解析するために使用されるアルゴリズムを設定します。
これをオフにするには `false` を使用し、それ以外の場合は [commondir](https://www.npmjs.com/package/commondir) モジュールを使用します。

`opts.fullPaths` は、モジュールIDを数値インデックスに変換することを無効にします。
これは、バンドルが生成された元のパスを保持するのに便利です。

`opts.builtins` は、使用する組み込み関数のリストを設定します。
デフォルトでは、このディストリビューションの `lib/builtins.js` に設定されています。

`opts.bundleExternal` は外部モジュールをバンドルするかどうかを設定する真偽値オプションです。
デフォルトは `true` です。

`opts.browserField` が `false` の場合、 `package.json` の `browser` フィールドは無視されます。

`opts.insertGlobals` が `true` の場合は、より速いビルドで大きな出力バンドルのために AST を解析することなく、常に `process`、 `global`、 `__filename`、 および `__dirname` を挿入します。
デフォルトは `false` です。

`opts.detectGlobals` が `true` の場合、必要に応じて定義された `process`、 `global`、 `__filename`、および `__dirname` のすべてのファイルをスキャンします。
このオプションを使用すると、npm モジュールが動作する可能性は高くなりますが、バンドルには時間がかかります。 デフォルトは `true` です。

`opts.ignoreMissing` が `true` の場合、何にも解決されない `require()` 文は無視されます。

`opts.debug` が `true` の場合、バンドルの最後にソースマップをインラインで追加します。
これにより、最新のブラウザを使用している場合にすべてのオリジナルファイルを表示できるため、デバッグが容易になります。

`opts.standalone` が空でない文字列である場合、その名前と [umd](https://github.com/forbeslindesay/umd) ラッパーを持つスタンドアロンモジュールが作成されます。文字列名に `.` をセパレータとして使用して、スタンドアロンのグローバルエクスポートで名前空間を使用できます（例： `'A.B.C'`）。
グローバルなエクスポートは[サニタイズとキャメルケース化](https://github.com/ForbesLindesay/umd#name-casing-and-characters)されます。

スタンドアローンモードでは、元のソースからの `require()` 呼び出しがまだ周りにあり、AMDローダーが `require()` 呼び出しをスキャンする可能性があります。
これらのコールを [derequire](https://www.npmjs.com/package/derequire) で削除することができます。

```bash
$ npm install -g derequire
$ browserify main.js --standalone Foo | derequire > bundle.js
```

`opts.insertGlobalVars` は、 `opts.vars` パラメーターとして `insert-module-globals` に渡されます。

`opts.externalRequireName` は、デフォルトの `expose` モードで `'require'` に設定されていますが、別の名前を使用することもできます。

`opts.bare` は Node 組み込み関数を含まないバンドルを作成し、 `__dirname` と `__filename` を除くグローバルノード変数を置き換えません。

`opts.node` は Node で実行され、依存関係のブラウザ・バージョンを使用しないバンドルを作成します。
`{bare：true、browserField：false}` を渡すのと同じです。

ファイルに javascript ソースコードが含まれていない場合は、対応するトランスフォームも指定する必要があります。

他のすべてのオプションは [module-deps](https://www.npmjs.com/package/module-deps) および [browser-pack](https://www.npmjs.com/package/browser-pack) に直接転送されます。


### b.add(file, opts)

バンドルがロードされたときに実行される `file` からエントリファイルを追加します。

`file` が配列の場合、`file` 内の各項目はエントリファイルとして追加されます。


### b.require(file, opts)

バンドルの外側から `require(file)` で `file` を利用可能にする。

`file` パラメータは、`node_modules` のファイルを含め、 `require.resolve()` で解決できるものです。 `require.resolve()` と同様に、`file` に `./` という接頭語を付けて、ローカルファイルを要求する必要があります（ `node_modules` ではなく）。

`file` もストリームにすることができますが、 `opts.basedir` も使用して相対的な要求を解決できるようにする必要があります。

`file` が配列の場合、ファイル内の各項目が必要になります。 
`file` 配列形式では、各項目に文字列またはオブジェクトを使用できます。
オブジェクト項目には `file` プロパティがあり、残りのパラメータは `opts` に使用されます。

`opts` の `expose` プロパティを使用して、カスタム依存関係名を指定します。
`require('./vendor/angle/angular.js'、{expose： 'angular'})` は `require('angular')` を有効にします。


### b.bundle(cb)

ファイルとその依存関係を1つの javascript ファイルにバンドルします。

読み込み可能なストリームを javascript ファイルの内容で返します。
またはオプションで `cb(err、buf)` を指定してバッファリングされた結果を取得します。


### b.external(file)

現在のバンドルに `file` がロードされないようにします。
代わりに別のバンドルから参照します。

`file` が配列の場合、`file` の各項目は外部化されます。

`file` が別のバンドルの場合、そのバンドルの内容は読み込まれ、`file` のバンドルがバンドルされると現在のバンドルから除外されます。


### b.ignore(file)

`file` のモジュール名またはファイルが出力バンドルに表示されないようにします。

`file` が配列の場合、 `file` の各項目は無視されます。

代わりに `module.exports = {}` でファイルを取得します。


### b.exclude(file)

`file` のモジュール名またはファイルが出力バンドルに表示されないようにします。

`file` が配列の場合、`file` の各項目は除外されます。

コードがそのファイルを `require()` しようとすると、それをロードするための別のメカニズムを提供していなければ、それはスローされます。


### b.transform(tr, opts={})

変換関数またはモジュール名 `tr` で `require()` 呼び出しのために解析する前に、ソースコードを変換してください。

`tr` が関数の場合、 `tr(file)` で呼び出され、生ファイルの内容を取り込み、変換されたソースを生成する[スルーストリーム](https://github.com/substack/stream-handbook#through)を返します。

`tr` が文字列の場合は、[変換モジュール](https://github.com/browserify/module-deps#transforms)のモジュール名またはファイルパスで、次のシグネチャを持つ必要があります。

```javascript
var through = require('through');
module.exports = function (file) { return through() };
```


必ず[スルーモジュール](https://www.npmjs.com/package/through)を使用する必要はありません。
Browserify は、ノードv0.10に組み込まれた、より新しい、より冗長な [Transform ストリーム](http://nodejs.org/api/stream.html#stream_class_stream_transform_1)と互換性があります。

```javascript
var coffee = require('coffee-script');
var through = require('through');

b.transform(function (file) {
    var data = '';
    return through(write, end);

    function write (buf) { data += buf }
    function end () {
        this.queue(coffee.compile(data));
        this.queue(null);
    }
});
```

コマンドラインで `-c` フラグを指定すると、次のことができます。

```bash
$ browserify -c 'coffee -sc' main.coffee > bundle.js
```

または、より良い方法として、[coffeeify モジュール](https://github.com/jnordberg/coffeeify)を使用します。

```bash
$ npm install coffeeify
$ browserify -t coffeeify main.coffee > bundle.js
```

`opts.global` が `true` の場合、変換は `node_modules/` ディレクトリにレベルが存在するかどうかにかかわらず、すべてのファイルに対して動作します。
ほとんどの場合、通常の変換で十分なので、グローバル変換は慎重かつ控えめに使用してください。
また、通常の変換でできるように、 `package.json` でグローバル変換を構成することもできません。

グローバル変換は、通常の変換が実行された後に常に実行されます。

Transformsは、[subarg](https://www.npmjs.com/package/subarg) 構文を使用してコマンドラインからオプションを取得できます。

```bash
$ browserify -t [ foo --bar=555 ] main.js
```

またはapiから…

```javasscript
b.transform('foo', { bar: 555 })
```

いずれの場合も、これらのオプションは変換関数の第2引数として提供されます。

```javascript
module.exports = function (file, opts) { /* opts.bar === 555 */ }
```

browserify コンストラクタに送信されるオプションも `opts._flags` の下にあります。
これらの browserify オプションは、browserify がデバッグモードで実行されているときにトランスフォームが何か異なる処理を行う必要がある場合などに必要になることがあります。


### b.plugin(plugin, opts)

`opts` に `plugin` を登録します。
プラグインは、文字列モジュール名または変換と同じ関数にすることができます。

`plugin(b、opts)` が browserify `b` インスタンスとともに呼び出されます。

詳細については、下記のプラグインのセクションを参照してください。


### b.pipeline

これらのラベルを持つ内部[ラベル付きストリーム・スプライサ](https://www.npmjs.com/package/labeled-stream-splicer)パイプラインがあります。

- `'record'` - 後続の `bundle()` 呼び出しで後で再生するための入力を保存する
- `'deps'` - module-deps
- `'json'` - jsonファイルの先頭に `module.exports =` を追加する
- `'unbom'` - バイトオーダーマーカーを削除する
- `'unshebang'` - 最初の行の `＃!` ラベルを取り除く
- `'syntax'` - 構文エラーをチェックする
- `'sort'` - 確定的なバンドルの依存関係をソートする
- `'dedupe'` - 重複するソースコンテンツを削除する
- `'label'` - ファイルに整数ラベルを適用する
- `'emit-deps'` - `'dep'` イベントを発行する
- `'debug'` - ソースマップを適用する
- `'pack'` - ブラウザパック
- `'wrap'` - 最終ラップを適用する、 `require=' と改行とセミコロン

`b.pipeline.get()` をラベル名で呼び出して、ストリームパイプラインのハンドルを取得し、 `push()`、 `unshift()` 、または `splice()` を使用して独自の変換ストリームを挿入できます。


### b.reset(opts)

パイプラインを通常の状態に戻します。 この関数は、 `bundle()` が複数回呼び出されたときに自動的に呼び出されます。

この関数は、  `'reset'` イベントをトリガします。


## package.json

browserify は、node と同様に、モジュール解決アルゴリズムで `package.json` を使用します。 `"main" `フィールドがある場合、browserify はその時点でパッケージの解決を開始します。 
`"main"` フィールドがない場合、browserify はモジュールルートディレクトリの `"index.js"` ファイルを探します。
`package.json` で行うことができるより洗練されたものがいくつかあります。


### browser field

ここでは特別な  ["browser"](https://gist.github.com/4339901) フィールドがあります。
このフィールドは、モジュールごとに `package.json` で設定して、ブラウザ固有のバージョンのファイルのファイル解決を上書きすることができます。

たとえば、`"main"` フィールドのブラウザ固有のモジュールエントリポイントを使用する場合は、`"browser"` フィールドを文字列に設定するだけです。

```json
"browser": "./browser.js"
```

ファイル単位で上書きすることもできます。

```json
"browser": {
  "fs": "level-fs",
  "./lib/ops.js": "./browser/opts.js"
}
```

ブラウザフィールドはローカルモジュールのファイルにのみ適用され、変換と同様に、`node_modules` ディレクトリには適用されません。


### browserify.transform

ソース変換は、 `package.json` の `browserify.transform` フィールドで指定できます。
ソース変換が `package.json` の [module-deps readme](https://github.com/browserify/module-deps#transforms) でどのように機能するかについて、より多くの情報があります。

たとえば、モジュールに [brfs](https://www.npmjs.com/package/brfs) が必要な場合は…

```json
"browserify": { "transform": [ "brfs" ] }
```

`package.json` に。 誰かがあなたのモジュールを `require()` しようとすると、あなたのモジュールを使っている人の明示的な介入なしに、あなたのモジュール内のファイルに `brfs` が自動的に適用されます。
`package.json` 依存関係フィールドに `transform` を必ず追加してください。


## events


### b.on('file', function (file, id, parent) {})
### b.pipeline.on('file', function (file, id, parent) {})

ファイルがバンドルに対して解決されると、バンドルは `file` パス全体、`require()` に渡された `id` 文字列、および [browser-resolve](https://github.com/defunctzombie/node-browser-resolve) によって使用される親オブジェクトを含む `'file'` イベントを出力します。

ファイルイベントが発生すると、ファイルウォッチャーを実装して、ファイルが変更されたときにバンドルを再生成することができます。


### b.on('package', function (pkg) {})
### b.pipeline.on('package', function (pkg) {})

パッケージファイルが読み込まれると、このイベントは内容で起動します。 パッケージディレクトリは、 `pkg.__ dirname` にあります。


### b.on('bundle', function (bundle) {})

`.bundle()` が呼び出されると、このイベントは `bundle` 出力ストリームで起動します。


### b.on('reset', function () {})

`.reset()` メソッドが呼び出されるか、 `.bundle()` の別の呼び出しによって暗黙的に呼び出されると、このイベントが発生します。


### b.on('transform', function (tr, file) {})
### b.pipeline.on('transform', function (tr, file) {})

変換がファイルに適用されると、 `'transform'` イベントは変換ストリーム `tr` と変換が適用されている `file` とともにバンドルストリームで起動します。


## plugins

より高度なユースケースの場合、変換は十分に拡張可能ではありません。 プラグインは、バンドルインスタンスを最初のパラメータとして、オプションハッシュを2番目のパラメータとして使用するモジュールです。

プラグインを使用して、変換できないファンシーな機能を実行できます。
たとえば、 [factor-bundle](https://www.npmjs.com/package/factor-bundle) は、複数のエントリポイントからの共通の依存関係を共通のバンドルに分解できるプラグインです。
`-p` と一緒にプラグインを使用し、[subarg](https://www.npmjs.com/package/subarg) 構文でプラグインにオプションを渡します。

```bash
browserify x.js y.js -p [ factor-bundle -o bundle/x.js -o bundle/y.js ] \
  > bundle/common.js
```

プラグインのリストについては、npm の [browserify-plugin タグ](https://www.npmjs.com/browse/keyword/browserify-plugin)を参照してください。


## list of source transforms

[既知の browserify 変換をリストするwikiページ](https://github.com/browserify/browserify/wiki/list-of-transforms)があります。

トランスフォームを作成する場合は、そのwikiページにトランスフォームを追加し、 `browserify-transform` の `package.json` キーワードを追加して、`npmjs.org` のすべての[browserifyトランスフォームをブラウズ](https://www.npmjs.com/browse/keyword/browserify-transform)できるようにしてください。


## third-party tools

[既知のブラウザツールをリストしたwikiページ](https://github.com/browserify/browserify/wiki/browserify-tools)があります。

ツールを書く場合は、その wiki ページに追加して、 `browserify-tool` の `package.json` キーワードを追加して、 `npmjs.org` のすべての [browserifyツールをブラウズ](https://www.npmjs.com/browse/keyword/browserify-tool) できるようにしてください。


## changelog

リリースは [changelog.markdown](https://github.com/browserify/browserify/blob/master/changelog.markdown) と [browserify twitterフィード](https://twitter.com/browserify) に記載されています。


## license

[MIT](https://github.com/browserify/browserify/blob/master/LICENSE)

![browserify](https://github.com/browserify/browserify/raw/master/assets/browserify.png)

