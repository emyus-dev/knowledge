# Browserify-Handbook

[@see](https://github.com/browserify/browserify-handbook)

## introduction

このドキュメントでは、[browserify](http://browserify.org/) を使用してモジュラーアプリケーションを構築する方法について説明します。

browserify は、ブラウザ用の [node1-flavored](http://nodejs.org/docs/latest/api/modules.html)の commonjs モジュールをコンパイルするためのツールです。

npm でパッケージをバンドルしてインストールする以外は、他の容量で [node](http://nodejs.org/) 自体を使用しない場合でも、browserify を使用してコードを整理し、サードパーティのライブラリを使用することができます。

ブラウザで使用するモジュールシステムは node と同じです。したがって、[npm](https://npmjs.org/) に公開されたパッケージでもともとは node ではなくブラウザで使用することを意図していたものは、ブラウザで正常に動作します。

人々は npm にモジュールを公開しています。これらのモジュールは、browserify を使ってノードとブラウザの両方で動作するように意図的に設計されており、npm の多くのパッケージはブラウザだけで使用することを意図しています。 [npm はすべての javascript](http://maxogden.com/node-packaged-modules.html)、front、backend に似ています。

## node packaged manuscript

このハンドブックは、適切に npm でインストールすることができます。

```bash
npm install -g browserify-handbook
```

これで、あなたの $PAGER にこの readme ファイルを開く `browserify-handbook` コマンドがあります。それ以外の場合は、現在行っているようにこの文書を読み続けることができます。

## node packaged modules

browserify の使い方や動作の仕方を深く理解する前に、まず commonjs モジュールシステムの [node-flavored バージョン](http://nodejs.org/docs/latest/api/modules.html) がどのように機能するかを理解することが重要です。

### require

node には、他のファイルからコードをロードするための `require()` 関数があります。

[npm](https://npmjs.org/) でモジュールをインストールする場合…

```bash
npm install uniq
```

次に、`nums.js` ファイルで、`require('uniq')` を使うことができます。

```javascript
var uniq = require("uniq");
var nums = [5, 2, 1, 3, 2, 5, 4, 2, 0, 1];
console.log(uniq(nums));
```

このプログラムを node とともに実行すると、次のように出力されます。

```bash
$ node nums.js
[ 0, 1, 2, 3, 4, 5 ]
```

相対ファイルを要求するには、`..` で始まる文字列を要求します。たとえば、 `main.js` からファイル `foo.js` をロードするには、`main.js` で次のようにします。

```javascript
var foo = require("./foo.js");
console.log(foo(4));
```

`foo.js` が親ディレクトリにある場合は、代わりに `../foo.js` を使用できます。

```javascript
var foo = require("../foo.js");
console.log(foo(4));
```

または他の種類の相対的な経路についても同様です。  
相対パスは、呼び出し元のファイルの場所に関して常に解決されます。

`require()` は関数を返し、その戻り値を `uniq` という変数に代入したことに注意してください。  
他の名前を選ぶこともできましたし、同じ名前で働くこともできました。 `require()` は、指定したモジュール名のエクスポートを返します。

どのように `require()` が他の多くのモジュールシステムとは違うのですか？  
インポートは、グローバルなものやファイルローカルレキシカルのようなもので、他のモジュール自体で宣言されたものとは異なります。
`require()` でコードをインポートする node スタイルでは、プログラムを読んでいる誰かが、それぞれの機能がどこから来たのかを簡単に知ることができます。  
このアプローチは、アプリケーション内のモジュールの数が増えるにつれて大幅に向上します。

### exports

他のファイルがインポートできるようにファイルから 1 つのものをエクスポートするには、 `module.exports` の値を割り当てます。

```javascript
module.exports = function(n) {
  return n * 111;
};
```

ここで、いくつかのモジュール `main.js` が `foo.js` をロードすると、 `require('./ foo.js')` の戻り値がエクスポートされた関数になります。

```javascript
var foo = require("./foo.js");
console.log(foo(5));
```

のプログラムは以下を出力します。

```bash
555
```

関数だけでなく、 `module.exports` を使用してあらゆる種類の値をエクスポートすることができます。

たとえば、これは完全にうまくいきます。

```javascript
module.exports = 555;
```

そうです。

```javascript
var numbers = [];
for (var i = 0; i < 100; i++) numbers.push(i);

module.exports = numbers;
```

アイテムをオブジェクトにエクスポートするためのエクスポートを行う別の形式があります。  
ここでは、 `module.exports` の代わりに `exports` が使用されます。

```javascript
exports.beep = function(n) {
  return n * 1000;
};
exports.boop = 555;
```

このプログラムは次のものと同じです。

```javascript
module.exports.beep = function(n) {
  return n * 1000;
};
module.exports.boop = 555;
```

`module.exports` は `exports` と同じで、最初は空のオブジェクトに設定されているためです。

ただし、次のことはできません。

```javascript
// これは動きません。
exports = function(n) {
  return n * 1000;
};
```

`exports` 値は `module` オブジェクト上に存在するため、`module.exports` の代わりに新しい値を代入すると元の参照がマスクされるためです。

代わりに、単一のアイテムをエクスポートする場合は、常に次の操作を行います。

```javascript
// 代わりに
module.exports = function(n) {
  return n * 1000;
};
```

まだ混乱している場合は、モジュールがバックグラウンドでどのように動作するかを理解してください。

```javascript
var module = {
  exports: {}
};

// モジュールが必要な場合は、基本的には関数にラップされています
(function(module, exports) {
  exports = function(n) {
    return n * 1000;
  };
})(module, module.exports);

console.log(module.exports); // まだ空のオブジェクトです :(
```

たいていの場合、`module.exports` を使って単一の関数またはコンストラクタをエクスポートしたいのは、モジュールが 1 つのことを行うのが最善であるからです。

`exports` 機能はもともと機能をエクスポートする主な方法であり、`module.exports` は補足的なものでしたが、`module.exports` はより直接的で明確で重複を避けるために実際にはもっと便利でした。

初期には、このスタイルはずっと一般的でした。

foo.js:

```javascript
exports.foo = function(n) {
  return n * 111;
};
```

main.js:

```javascript
var foo = require("./foo.js");
console.log(foo.foo(5));
```

しかし、`foo.foo` はちょっと余計なことです。 `module.exports` を使用すると、より明確になります。

foo.js:

```javascript
module.exports = function(n) {
  return n * 111;
};
```

main.js:

```javascript
var foo = require("./foo.js");
console.log(foo(5));
```

### bundling for the browser

node でモジュールを実行するには、どこかから始める必要があるでしょうか。

node では、`node` コマンドにファイルを渡してファイルを実行します。

```bash
$ node robot.js
beep boop
```

browserify では、これと同じことを行いますが、ファイルを実行する代わりに、 `>` 演算子を使用してファイルに書き込むことができる標準出力に連結された javascript ファイルのストリームを生成します。

```bash
$ browserify robot.js > bundle.js
```

ここで `bundle.js` には、 `robot.js` が動作するために必要な javascript がすべて含まれています。 それをいくつかの html で 単一の `script` タグに入れてください。

```html
<html>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```

おまけ：スクリプトタグを `</body>` の直前に置くと、dom の onready イベントを待たずに、ページ上のすべての dom 要素を使用できます。

バンドルにはもっと多くのことがあります。 この文書の他の場所にあるバンドルのセクションを参照してください。

### how browserify works

Browserify は、ソースコードの[抽象構文ツリー](https://en.wikipedia.org/wiki/Abstract_syntax_tree)の[静的解析](http://npmjs.org/package/detective)を使用して見つかった `require()` 呼び出しを、入力したエントリポイントファイルから検索します。

`require()` 呼び出しで文字列を指定すると、browserify はこれらのモジュール文字列をファイルパスに解決し、d 依存性グラフ全体が訪れるまで `require()` 呼び出しのファイルパスを再帰的に検索します。

各ファイルは、静的に解決された名前を内部 ID にマップする最低限の `require()` 定義を持つ単一の javascript ファイルに連結されます。

つまり、生成するバンドルは完全に自己完結型であり、アプリケーションの作業に必要なものはすべて無視できないほどのオーバーヘッドがあります。

browserify の動作の詳細については、このドキュメントのコンパイラパイプラインのセクションを参照してください。

### how node_modules works

node には、ライバルのプラットフォーム間でユニークなモジュールを解決するための巧妙なアルゴリズムがあります。

`$PATH` がコマンドライン上でどのように動作するかといったシステム検索パスの配列からパッケージを解決する代わりに、node のメカニズムはデフォルトではローカルです。

`/beep/boop/bar.js` から `('./foo.js')` が必要な場合、node は `/beep/boop/foo.js` の `./foo.js` を探します。 `./` または `../` で始まるパスは、 `require()` を呼び出すファイルに対して常にローカルです。

しかし、 `/beep/boop/foo.js` からの `require('xyz')` のような非相対的な名前が必要な場合、node はこれらのパスを順番に検索し、最初の一致で停止し、何も見つからない場合はエラーを発生させます。

```
/beep/boop/node_modules/xyz
/beep/node_modules/xyz
/node_modules/xyz
```

存在する `xyz` ディレクトリごとに、node はまず `xyz/package.json` を探し、 `"main"` フィールドが存在するかどうかを調べます。 `"main"` フィールドは、ディレクトリパスを `require()` するとどのファイルを担当するかを定義します。

たとえば、`/beep/node_modules/xyz` が最初に一致し、`/beep/node_modules/xyz/package.json` が次のようになっているとします。

```json
{
  "name": "xyz",
  "version": "1.2.3",
  "main": "lib/abc.js"
}
```

`/beep/node_modules/xyz/lib/abc.js` からのエクスポートは `require('xyz')`によって返されます。

`package.json` も `"main"` フィールドもない場合は、 `index.js` が仮定されます。

```
/beep/node_modules/xyz/index.js
```

必要な場合は、パッケージに入って特定のファイルを選ぶことができます。  
たとえば、 `lib/clone.js` ファイルを `dat` パッケージからロードするには、次のようにします。

```javascript
var clone = require('dat/lib/clone.js')
```

再帰的な node_modules 解決では、最初の `dat` パッケージがディレクトリ階層の上にあることがわかります。その後 `lib/clone.js` ファイルがそこから解決されます。 これは `require('dat/lib/clone.js')` のアプローチは `require('dat')` 出来る場所であれば動作します。

node にはパスの配列を検索するためのメカニズムもありますが、このメカニズムは推奨されていません。そうしないと非常に良い理由がない限り、 `node_modules/` を使用する必要があります。

node のアルゴリズムと npm がパッケージをインストールする方法の素晴らしい点は、ほとんどすべての他のプラットフォームとは異なり、バージョンの競合がないことです。 npm は各パッケージの依存関係を `node_modules` にインストールします。

各ライブラリは、依存関係が格納されている独自のローカル `node_modules/` ディレクトリを取得し、各依存関係の依存関係には独自の `node_modules/` ディレクトリがあります。

これは、パッケージが同じアプリケーション内の異なるバージョンのライブラリをうまく使用できることを意味します。これにより、APIの反復処理に必要な調整オーバーヘッドが大幅に削減されます。 この機能は、npm などのエコシステムにとって非常に重要です。ここでは、パッケージの公開方法と組織方法を管理する中心的な権限はありません。 すべての人は、適切に見えるように公開するだけで、依存関係のバージョンの選択が同じアプリケーションに含まれる他の依存関係にどのように影響するか心配する必要はありません。

`node_modules/` がどのように独自のローカルアプリケーションモジュールを構成するかを活用することができます。詳細については、`avoiding ../../../../../../..` セクションを参照してください。

### why concatenate

Browserify は、サーバー上で実行されるビルドステップです。 その中にすべてがある単一のバンドルファイルを生成します。

ブラウザ用のモジュールシステムを実装する他の方法と、その長所と短所を次に示します。

#### window globals

 モジュールシステムの代わりに、各ファイルは `window` グローバルオブジェクトのプロパティを定義するか、内部の名前空間スキームを作成します。

この新しいアプローチでは、アプリケーションがレンダリングされるすべての html ページに新しい `<script>` タグが必要になるため、このアプローチは徹底的にはうまく機能しません。  
さらに、環境内に既にグローバルが存在すると予想される他のファイルよりも前に、いくつかのファイルをインクルードする必要があるため、ファイルは非常に順序が敏感です。

このように構築されたアプリケーションをリファクタリングまたはメンテナンスすることは困難です。 プラスの面では、すべてのブラウザがこのアプローチをネイティブにサポートしているため、サーバー側のツールが必要になりません。

各 `<script>` タグが新しい往復HTTP要求を開始するので、このアプローチは非常に遅くなる傾向があります。

#### concatenate

`window` グローバルの代わりに、すべてのスクリプトがサーバー上で事前に連結されています。 コードは依然としてオーダセンシティブで、メンテナンスは難しいですが、単一の `<script>` タグに対する単一のHTTPリクエストのみを実行する必要があるため、ロードがずっと高速になります。

#### AMD

`<script>` タグを使用する代わりに、すべてのファイルは `define()` 関数とコールバックでラップされます。 これは[AMD](http://requirejs.org/docs/whyamd.html)です。

最初の引数は、コールバックに渡された各引数にそのマップをロードするためのモジュールの配列です。 すべてのモジュールがロードされると、コールバックが起動します。

```javascript
define(['jquery'] , function ($) {
    return function () {};
});
```

最初の引数にモジュールに名前を付けることで、他のモジュールにも含めることができます。

各コールバックを文字列化し、[regexp](https://github.com/requirejs/requirejs/blob/57c48253e42133a61075da67809b91ea34f89811/require.js#L16) で `require()` 呼び出しをスキャンする commonjs の糖衣構文があります。

このように記述されたコードは、順序が明示的な依存関係情報によって解決されるため、連結またはグローバルよりも順序の影響を受けません。

パフォーマンス上の理由から、ほとんどの時間 AMD はサーバ側で1つのファイルにまとめられており、開発中は AMD の非同期機能を実際に使用する方が一般的です。

#### bundling commonjs server-side

パフォーマンスのための構築ステップと便宜のための砂糖構文を使用する場合は、AMD のビジネス全体を一掃し、commonjs をバンドルしてみませんか？ ツーリングを使用すると、モジュールを注文感度に対応させることができ、開発環境と製造環境がはるかに似ていて脆弱になりません。 CJS 構文はより良く、ノードと npm のために生態系が爆発的になります。

node とブラウザの間でコードをシームレスに共有できます。 ソースマップと自動再構築のためのビルドステップとツールが必要です。

さらに、ノードのモジュールルックアップアルゴリズムを使用して、バージョンの不一致の問題から私たちを救うことができます。これにより、同じアプリケーション内で必要なパッケージの複数の競合するバージョンを持つことができます。 バイトをワイヤの下に保存するには、この文書の別の部分で説明している重複排除が可能です。


## development

連結にはいくつかの欠点がありますが、これらは開発ツールを使用して適切に対応することができます。

### source maps

Browserify は、ソースマップを有効にするために `--debug/-d` フラグと `opts.debug` パラメータをサポートしています。  
ソースマップは、バンドルファイルにスローされた例外の行と列のオフセットを元のソースのオフセットとファイル名に戻すようブラウザに指示します。

ソースマップには元のファイルの内容がすべてインラインで含まれているため、バンドルファイルを Web サーバーに配置するだけで、パスが正しく設定された Web サーバーからすべての元のソースコンテンツにアクセスできるようにする必要はありません。

#### exorcist

すべてのソースファイルをインラインソースマップにインライン展開することの欠点は、バンドルが2倍大きいことです。 これはローカルでデバッグするのには問題ありませんが、ソースマップを実稼働環境に出荷するためには実用的ではありませ ただし、エクソシストを使用して、インラインソースマップを個別の `bundle.map.js` ファイルに引き出すことができます。

```bash
browserify main.js --debug | exorcist bundle.map.js > bundle.js
```

### auto-recompile

毎回バンドルを再コンパイルするコマンドを実行すると、遅く退屈になることがあります。 幸いにも、この問題を解決するための多くのツールがあります。 これらのツールの中には、さまざまなレベルのライブリロードをサポートするものと、より伝統的な手動リフレッシュサイクルを持つものがあります。

これらはあなたが使用できるツールのほんの一部ですが、npm にはさらに多くのものがあります！ ここにはさまざまなトレードオフや開発スタイルを含むさまざまなツールがあります。 あなた自身の個人的な期待や経験と最も強く共鳴するツールを見つけるために少し前から作業することができますが、この多様性はプログラマーがより効果的になり、創造性と実験の余地が増えると考えています。 
ツーリングの多様性と 最小限のbrowserify のコアは、中長期的に見ると browserify のコア(有意義なバージョン管理とコアのビットロットであらゆる種類の混乱を引き起こします)にいくつかの "勝者" を選ぶよりも健康的だと思います。

つまり、ブラウザ対応の開発ワークフローを設定する際に考慮したいモジュールがいくつかあります。 しかし、このリストに載っていない他のツールを見てみましょう！


#### [watchify](https://npmjs.org/package/watchify)

watchify は browserify と同じ意味で使用できますが、出力ファイルに一度書き込むのではなく、watchify はバンドルファイルを書き込んだ後、依存関係グラフのすべてのファイルを監視して変更を監視します。  
ファイルを変更すると、新しいバンドルファイルは積極的なキャッシングのためにはじめよりも速く書き込まれます。

`-v` を使用すると、新しいバンドルが書き込まれるたびにメッセージを出力できます。

```bash
$ watchify browser.js -d -o static/bundle.js -v
610598 bytes written to static/bundle.js  0.23s
610606 bytes written to static/bundle.js  0.10s
610597 bytes written to static/bundle.js  0.14s
610606 bytes written to static/bundle.js  0.08s
610597 bytes written to static/bundle.js  0.08s
610597 bytes written to static/bundle.js  0.19s
```

以下は、 `package.json` の `"scripts"` フィールドで watchify と browserify を使用するための便利な設定です。

```json
{
  "build": "browserify browser.js -o static/bundle.js",
  "watch": "watchify browser.js -o static/bundle.js --debug --verbose",
}
```

プロダクション用のバンドルをビルドするには、npm はビルドを実行し、開発中は watch を実行するためにファイルを監視します。

npm の詳細については[こちら](http://substack.net/task_automation_with_npm_run)をご覧ください。


#### [beefy](https://www.npmjs.org/package/beefy)

コードを変更したときにコードを自動的に再コンパイルする Web サーバーを起動させる場合は、[beefy](https://www.npmjs.org/package/beefy) をチェックしてください。

beefy にファイルを渡すだけです。

```bash
beefy main.js
```

http ポートにショップを設定します。


#### [wzrd](https://github.com/maxogden/wzrd)

beefy と同じような目的で、より最小限の形では [wzrd](https://github.com/maxogden/wzrd) です。

`npm install -g wzrd` するだけです。

```bash
wzrd app.js
```

ブラウザーで [http://localhost:9966/](http://localhost:9966/) を開きます。


#### browserify-middleware, enchilada

express を使用している場合は、 [browserify-middleware](https://www.npmjs.org/package/browserify-middleware) または [enchilada](https://www.npmjs.org/package/enchilada) をチェックしてください。

どちらも、ブラウザーバンドルを提供するための express アプリケーションにドロップできるミドルウェアを提供します。


#### [livereactload](https://github.com/milankinen/livereactload)

livereactloadは、コードを修正するときにWebページの状態を自動的に更新する [react](https://github.com/facebook/react) のツールです。

livereactload は、 `-t livereactload` で読み込むことのできる通常のブラウザ用変換ですが、詳細は[プロジェクトの readme](https://github.com/milankinen/livereactload#livereactload) を参照してください。


#### [browserify-hmr](https://github.com/AgentME/browserify-hmr)

browserify-hmr はホットモジュール置き換え（hmr）を行うためのプラグインです。

ファイルは更新を受け入れるものとしてマークすることができます。 自分自身の更新を受け入れるファイルを変更した場合、または更新を受け入れるファイルの依存関係を変更した場合、ファイルは新しいコードで再実行されます。

たとえば、ファイルがある場合、`main.js`：

```javascript
document.body.textContent = require('./msg.js')

if (module.hot) module.hot.accept()
```
とファイル `msg.js`：

```javascript
module.exports = 'hey'
```

`main.js` で変更を監視し、browserify-hmr プラグインをロードすることができます。

```bash
$ watchify main.js -p browserify-hmr -o public/bundle.js -dv
```

静的ファイルの内容を `public/` 静的ファイルサーバーで提供します。

```bash
$ ecstatic public -p 8000
```

`http://localhost:8000/` をロードすると、ページ上にメッセージ `hey` が表示されます。


`msg.js` を次のように変更した場合：

```javascript
module.exports = 'wow'
```

数秒後にページが更新されて、それだけですべてが表示されます。

Browserify-HMR を [react-hot-transform](https://github.com/AgentME/react-hot-transform) とともに使用すると、 `module.hot` API を使用してコードに加えて、すべての React コンポーネントを自動的に更新できるようになります。 [livereactload](https://github.com/milankinen/livereactload) とは異なり、各変更時にバンドル全体ではなく、変更されたファイルだけが再実行されます。


#### [budo](https://github.com/mattdesl/budo)

Budo は、インクリメンタルバンドルと LiveReload の統合（CSSインジェクションを含む）に重点を置いたブラウザ対応開発サーバーです。

次のようにインストールします。

```bash
npm install budo -g
```

エントリーファイルでそれを実行してください。

```bash
budo app.js
```

これにより [http://localhost:9966/](http://localhost:9966/) のサーバーがデフォルトの `index.html` で起動し、ソースをファイル保管庫に段階的にバンドルします。
バンドルが完了するまで要求が遅れるので、更新中のページを更新すると古いバンドルや空のバンドルにはなりません。

LiveReload を有効にし、ブラウザで JS/HTML/CSS の変更を更新するには、次のように実行します。

```bash
budo app.js --live
```


### using the api directly

開発のために通常の `http.createServer()` から API を直接使用することもできます。

```javascript
var browserify = require('browserify');
var http = require('http');

http.createServer(function (req, res) {
    if (req.url === '/bundle.js') {
        res.setHeader('content-type', 'application/javascript');
        var b = browserify(__dirname + '/main.js').bundle();
        b.on('error', console.error);
        b.pipe(res);
    }
    else res.writeHead(404, 'not found')
});
```


### grunt

grunt を使用する場合は、おそらく [grunt-browserify](https://www.npmjs.org/package/grunt-browserify) プラグインを使用したいと思うでしょう。


### gulp

gulp を使用する場合は、browserify API を直接使用する必要があります。

ここは、[gulp と browserify を始めるためのガイド](https://www.viget.com/articles/gulp-browserify-starter-faq)です。

ここでは、公式の gulp のレシピから [gulp を使用して watchify browserify ビルドを高速化する方法](https://github.com/gulpjs/gulp/blob/master/docs/recipes/fast-browserify-builds-with-watchify.md)に関するガイドがあります。


## builtins

もともとブラウザで node 作業のために書き込まれた npm モジュールを増やすために、browserify は多くのブラウザ固有の node コアライブラリの実装を提供しています。

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

events、stream、url、path、および querystring は、ブラウザ環境で特に便利です。

さらに、browserify が `Buffer`、`process`、`global`、 `__filename`、または `__dirname` の使用を検出すると、ブラウザに適切な定義が含まれます。

だから、たとえモジュールがたくさんの buffer と stream 操作を行ったとしても、それはサーバIOを何もしない限り、おそらくブラウザで動作します。

以前に node を作成していない場合は、それぞれのグローバルができることのいくつかの例があります。 これらのグローバルは、あなたやあなたが依存しているモジュールがそれらを使用するときにのみ実際に定義されることにも注意してください。


### [Buffer](http://nodejs.org/docs/latest/api/buffer.html)

node では、すべてのファイルおよびネットワークAPIが Buffer チャンクを処理します。browserify では、Buffer API は [buffer](https://www.npmjs.org/package/buffer) によって提供されています。これは、古いブラウザーのフォールバックを伴う非常に効果的な方法で拡張型の配列を使用します。

Buffer を使って base64 文字列を16進数に変換する例を以下に示します。

```javascript
var buf = Buffer('YmVlcCBib29w', 'base64');
var hex = buf.toString('hex');
console.log(hex);
```

この例は次のように出力されます。

```
6265657020626f6f70
```


### [process](http://nodejs.org/docs/latest/api/process.html#process_process)

ノードでは、`process` は、環境、シグナル、および標準IOストリームなどの実行中のプロセスの情報と制御を処理する特別なオブジェクトです。

特に重要なのは、イベントループとインターフェースする `process.nextTick()` の実装です。

ブラウザでは、プロセスの実装は `process.nextTick()` とそれ以外のものを提供する [process モジュール](https://www.npmjs.org/package/process) によって処理されます。

`process.nextTick()` は次のようになります。

```javascript
setTimeout(function () {
    console.log('third');
}, 0);

process.nextTick(function () {
    console.log('second');
});

console.log('first');
```

このスクリプトは次のように出力します。

```
first
second
third
```

`process.nextTick(fn)` は `setTimeout(fn、0)` と似ていますが、互換性の理由から、javascript エンジンでは、 `setTimeout` が人為的に遅いため速くなります。


### [global](http://nodejs.org/docs/latest/api/all.html#all_global)

node では、`global` は、`window` がブラウザでどのように動作するかと同様に、グローバル変数が接続されるトップレベルのスコープです。 browserify では、 `global` は `window` オブジェクトの単なるエイリアスです。


### [__filename](http://nodejs.org/docs/latest/api/all.html#all_filename)

`__filenameは` 、現在のファイルへのパスです。ファイルごとに異なります。

システムパス情報を表示しないようにするには、このパスは `browserify()` に渡す `opts.basedir` をルートとします。デフォルトは[現在の作業ディレクトリ](https://en.wikipedia.org/wiki/Current_working_directory)です。

`main.js` がある場合：

```javascript
var bar = require('./foo/bar.js');

console.log('here in main.js, __filename is:', __filename);
bar();
```

と `foo/bar.js`：

```javascript
module.exports = function () {
    console.log('here in foo/bar.js, __filename is:', __filename);
};
```

`main.js` から browserify を実行すると、次の出力が得られます。

```bash
$ browserify main.js | node
here in main.js, __filename is: /main.js
here in foo/bar.js, __filename is: /foo/bar.js
```


### [__dirname](http://nodejs.org/docs/latest/api/all.html#all_dirname)

`__dirname` は、現在のファイルのディレクトリです。  `__filename` と同様に、 `__dirname` は `opts.basedir` をルートにしています。

`__dirname` の使い方の例を次に示します。

`main.js`:
```javascript
require('./x/y/z/abc.js');
console.log('in main.js __dirname=' + __dirname);
```

`x/y/z/abc.js`:
```javascript
console.log('in abc.js, __dirname=' + __dirname);
```

出力:
```bash
$ browserify main.js | node
in abc.js, __dirname=/x/y/z
in main.js __dirname=/
```


## transforms

すべてをサポートしてベーキングをブラウジングする代わりに、ソースファイルをインプレースで変換するために使用される柔軟な変換システムをサポートしています。

coffeescript やテンプレートで書かれたファイルを必要とすることができ、すべてが javascript にコンパイルされます。

たとえば、coffeescript を使用するには、[coffeeify](https://www.npmjs.org/package/coffeeify) トランスフォームを使用します。 最初に `npm install coffeeify` を実行して coffeeofy をインストールしたことを確認してください。

```bash
$ browserify -t coffeeify main.coffee > bundle.js
```

APIを使用する場合：

```javascript
var b = browserify('main.coffee');
b.transform('coffeeify');
```

もっとも重要な点は、ソースマップを `--debug` または `opts.debug` で有効にした場合、 `bundle.js` は例外を元の coffeescript のソースファイルに戻してマップすることです。 これは、Firebug や Chrome のインスペクタでデバッグするのに非常に便利です。

### writing your own

トランスフォームは単純なストリーミングインターフェイスを実装しています。 `$CWD` を `process.cwd()` に置き換えるトランスフォームを次に示します。

```javascript
var through = require('through2');

module.exports = function (file) {
    return through(function (buf, enc, next) {
        this.push(buf.toString('utf8').replace(/\$CWD/g, process.cwd()));
        next();
    });
};
```

変換関数は、現在のパッケージ内のすべての `file` に対して起動し、変換を実行する変換ストリームを返します。 ストリームは、元のファイルの内容をブラウジングしてブラウザに書き込まれ、新しい内容を取得するためにストリームから読み込みをブラウズします。

トランスフォームをファイルに保存するか、パッケージを作成して、`-t ./your_transform.js` を付けて追加するだけです。

ストリームの仕組みの詳細については、[ストリームハンドブック](https://github.com/substack/stream-handbook)を参照してください。


## package.json

### browser field

browserify は、パッケージの `package.json` に `"browser"` フィールドを定義して、browserify に `main` フィールドと個々のモジュールのルックアップをオーバーライドするよう指示します。

`main` エントリポイントが `main.js` の node で、ブラウザ固有のエントリポイントが `browser.js` のモジュールがある場合は、次の操作を実行できます。

```json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": "browser.js"
}
```

何かしらが `require('mypkg')` を node に要求すると、 `main.js` からのエクスポートを取得しますが、ブラウザで `require('mypkg')` を実行すると、`browser.js` からのエクスポートが取得されます。

ブラウザーであるかどうかにかかわらず、この方法で `"browser"` フィールドを使用すると、実行時にブラウザーであるかどうかを確認するよりも、 node またはブラウザーのどちらにあるかに基づいて異なるモジュールをロードすることができます 。 node とブラウザの両方に対する `require()` 呼び出しが同じファイル内にある場合、browserify の静的解析には、それらのファイルを使用するかどうかのすべてが含まれます。

文字列の代わりにオブジェクトとして `"browser"` フィールドを使って、さらに多くのことを行うことができます。

たとえば、`lib/` で単一のファイルをブラウザ固有のバージョンで置き換えるだけの場合は、次のようにします。

```json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "lib/foo.js": "lib/browser-foo.js"
  }
}
```

パッケージ内でローカルに使用されているモジュールを交換する場合は、次のようにします。

```json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "fs": "level-fs-browser"
  }
}
```

ブラウザのフィールドに値を `false` に設定することで、ファイルを無視して空のオブジェクトにコンテンツを設定することができます。

```json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "winston": false
  }
}
```
`browser` フィールドは、現在のパッケージにのみ適用されます。 設定したマッピングは、依存関係やその関連まで伝播しません。 この分離はモジュール同士を保護するために設計されているため、モジュールが必要なときにシステム全体の影響を心配する必要はありません。 同様に、依存関係グラフの遠方にあるモジュールにローカル構成がどのように悪影響を与えるかを心配する必要はありません。


### browserify.transform field

モジュールがパッケージの `browserify.transform` フィールドにロードされたときに自動的に適用されるようにトランスフォームを構成できます。 たとえば、この `package.json` を使用して [brfs](https://npmjs.org/package/brfs) トランスフォームを自動的に適用できます。

```json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browserify": {
    "transform": [ "brfs" ]
  }
}
```

`main.js` で行うことができます。

```javascript
var fs = require('fs');
var src = fs.readFileSync(__dirname + '/foo.txt', 'utf8');

module.exports = function (x) { return src.replace(x, 'zzz') };
```

`fs.readFileSync()` 呼び出しは、モジュールの消費者が知る必要のない brfs によってインライン化されます。 トランスフォーム配列で好きなだけ多くのトランスフォームを適用することができ、それらは順番に適用されます。

`"browser"` フィールドと同様に、 `package.json` で設定された変換は、同じ理由でローカルパッケージにのみ適用されます。

#### configuring transforms

変換によっては、コマンドラインで設定オプションが使用されることがあります。 `package.json` からこれらを適用するには、以下を行うことができます。

**on the command line**

```bash
browserify -t coffeeify \
           -t [ browserify-ngannotate --ext .coffee --bar ] \
           index.coffee > index.js
```

**in package.json**

```json
"browserify": {
  "transform": [
    "coffeeify",
    ["browserify-ngannotate", {"ext": ".coffee", "bar": true}]
  ]
}
```


## finding good modules

ブラウザで動作する npm の[良いモジュールを見つける](http://substack.net/finding_modules)ためのヒントをいくつか紹介します。

- npm でインストールできる。

- `require()` を使用した readme のコードスニペット - 一見すると、現在取り組んでいるものにライブラリを統合する方法を見てください

- 範囲と目的について非常に明確で狭いアイデアがある

- 他のライブラリに委任するときを知っています - あまりにも多くのことをやろうとしません

- ソフトウェアスコープ、モジュール性、および私が一般的に同意するインターフェース（コード/ドキュメントを非常に詳細に読むことよりも速いことが多い）についての意見を書いた著者によって書かれ、

- どのモジュールが評価されているライブラリに依存しているかを検査します - これは npm に公開されたモジュールのパッケージページに焼き付けられます

github 上の星数、プロジェクト活動、滑らかなランディングページなどの他の指標は信頼性が高くありません。


### module philosophy

人々は、便利なユーティリティスタイルのものをたくさんエクスポートすることは、プログラマーがコードを消費する主な方法だと思っていました。これは他のほとんどのプラットフォームでコードをエクスポートおよびインポートする主な方法であり、実際には npm でも持続します。

しかし、テーマに関連しているがセパレート可能な機能を1つのパッケージにまとめるというこの[キッチン・シンクの考え方](https://github.com/substack/node-mkdirp/issues/17)は、github　以前、npm　以前の時代の出版と発見の難しさのためのアーティファクトであるようです。

利便性の助けを借りて1つの場所にすべての機能をエクスポートしようとするモジュールには、2つの大きな問題があります。

機能を持っているパッケージは、[新しい機能が属しているものと属していないもの](https://github.com/jashkenas/underscore/search?q=%22special-case%22&ref=cmdform&type=Issues)について、時間を浪費しています。 この種のパッケージには、問題のドメインの明確な自然の境界はありません。それは、範囲が何であるかについて、[誰もが賢明な意見](http://david.heinemeierhansson.com/2012/rails-is-omakase.html)です。

Node、npm、browserify はそうではありません。 彼らは評判が良く、参加型であり、適合性、基準、または「ベストプラクティス」の名目で締め固めようとするよりも、不一致と新しいアイデアやアプローチのめまぐるしい普及を祝うことになります。

ガウスのぼかしを行う必要がある人は誰も考えません。
「うーん、一般的な数学、統計、画像処理、ユーティリティライブラリをチェックして、どれがガウスのぼかしをしているのかを見てみようと思っているのですが、stats2 や image-pack-utils や maths-extra か… 」
いいえ、これはありません。 それをやめなさい。 `npm search gaussian` とすぐに [ndarray-gaussian-filter](https://npmjs.org/package/ndarray-gaussian-filter)を見て、彼らが望むものを正確に実行して、誰かの無視された偉大な実用主義の雑草で迷子になるのではなく、実際の問題を続けます。


## organizing modules

### avoiding ../../../../../../..

アプリケーション内のすべてが公式の npm に正しく属しているわけではなく、プライベートな npm または git repoを設定するオーバーヘッドは、多くの場合まだかなり大きいです。 `../../../../../../../` 相対パスの問題を回避するためのいくつかのアプローチがあります。


#### symlink

最も簡単なことは、アプリケーションのルートディレクトリを `node_modules/` ディレクトリにシンボリックリンクすることです。

[シンボリックリンクは `windows` で動作する](http://www.howtogeek.com/howto/windows-vista/using-symlinks-in-windows-vista/)ことを知っていますか？

プロジェクトルートの `lib/` ディレクトリを `node_modules` にリンクするには、次のようにします。

```bash
ln -s ../lib node_modules/app
```

あなたのプロジェクトのどこからでも `lib/foo.js` を入手するために `require('app/foo.js')` を実行することで、`lib/` にファイルを要求することができます。


#### node_modules

npm から第三者モジュールをチェックインすることなく内部モジュールをチェックインする方法が明白ではないので、アプリケーション固有のモジュールを `node_module` に入れることに基本的には反対します。

答えは非常に簡単です！ `node_modules` を無視する `.gitignore` ファイルがある場合：

```
node_modules
```

内部アプリケーションモジュールごとに例外を `!` で追加できます。

```
node_modules/*
!node_modules/foo
!node_modules/bar
```

親がすでに無視されている場合は、サブディレクトリを解除することはできません。 したがって、 `node_modules` を無視する代わりに、 `node_modules/*` trick を使用して `node_modules` 内のすべてのディレクトリを無視してから、例外を追加することができます。

アプリケーションのどこにいても、非常に大きく壊れやすい相対パスを必要とせずに `('foo')` または `require('bar')` することができます。

たくさんのモジュールがあり、npm によってインストールされたサードパーティ製のモジュールとは別のものにしたい場合は、 `node_modules/app` などの `node_modules` のディレクトリの下にそれらを置くだけです。

```
node_modules/app/foo
node_modules/app/bar
```

これで、アプリケーションのどこからでも `require('app/foo')` または `require('app/bar')` できるようになりました。

`.gitignore` で `node_modules/app` の例外を追加するだけです。

```
node_modules/*
!node_modules/app
```

アプリケーションが `package.json` で構成された transforms を持っていた場合は、`node_modules/foo` または `node_modules/app/foo` コンポーネントディレクトリに独自の `transform` フィールドを持つ別の`package.json` を作成する必要があります。 これにより、モジュールがアプリケーションの構成変更に対してより堅牢になり、アプリケーション外でパッケージを単独で再利用する方が簡単になります。


#### custom paths

`$NODE_PATH` 環境変数を使用するか、 `opts.paths` を使用して node および browserify のディレクトリを追加してモジュールを検索する方法については、いくつかの話があります。

他のほとんどのプラットフォームとは異なり、 `$NODE_PATH` を持つシェルディレクトリ形式のパスディレクトリを使用することは、 `node_modules` ディレクトリを有効に使用するのと比べて、ノードではあまり適していません。

これは、アプリケーションがより厳密にランタイム環境構成に結合されているため、動く部品が増え、アプリケーションが正しく設定されている場合にのみ機能するためです。

node と browserify 両方ともにサポートしていますが、 `$NODE_PATH` の使用は避けてください。


### non-javascript assets

多くのことを行うために使うことができる [browserify ransforms](https://github.com/substack/node-browserify/wiki/list-of-transforms) がたくさんあります。 通常 transforms は、非javascript アセットをバンドルファイルに含めるために使用されます。


#### brfs

node とブラウザの両方で動作するあらゆる種類のアセットを含める1つの方法は brfs です。

brfs は静的解析を使用して `fs.readFile()` と `fs.readFileSync()` の結果をコンパイル時にソースの内容にコンパイルします。

たとえば、この `main.js`：

```javascript
var fs = require('fs');
var html = fs.readFileSync(__dirname + '/robot.html', 'utf8');
console.log(html);
```

brfs によって適用されると、次のようになります。

```javascript
var fs = require('fs');
var html = "<b>beep boop</b>";
console.log(html);
```

brfsを通過するとき。

これは、node とブラウザでまったく同じコードを再利用できるため、モジュールの共有とテストがはるかに簡単になるため便利です。

`fs.readFile()` と `fs.readFileSync()` は、node と同じ引数を受け取ります。これにより、インラインイメージアセットを base64 でエンコードされた文字列として簡単に組み込むことができます。

```javascript
var fs = require('fs');
var imdata = fs.readFileSync(__dirname + '/image.png', 'base64');
var img = document.createElement('img');
img.setAttribute('src', 'data:image/png;base64,' + imdata);
document.body.appendChild(img);
```

バンドルにインライン展開したい CSS がある場合は、 [insert-css](https://npmjs.org/package/insert-css) などのモジュールの助けを借りてそれを行うこともできます。

```javascript
var fs = require('fs');
var insertStyle = require('insert-css');

var css = fs.readFileSync(__dirname + '/style.css', 'utf8');
insertStyle(css);
```

このように CSS を挿入すると、npm で配布される小さな再利用可能なモジュールでうまく動作します。完全に含まれているため、npm で配布しますが、browserify を使用してアセットマネジメントにもっと包括的にアプローチするには、 [atomify](https://www.npmjs.org/package/atomify) と [parcelify](https://www.npmjs.org/package/parcelify) をチェックします。


#### hbsify
#### jadeify
#### reactify


### reusable components

これらのアイデアをコード構成についてまとめると、アプリケーションや他のアプリケーションで再利用できる再利用可能なUIコンポーネントを構築できます。

空のウィジェットモジュールの例です。

```javascript
module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = document.createElement('div');
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
};
```

便利な javascript コンストラクタのヒント：上記のような `this instanceof Widget` チェックを追加すると、`new Widget` または `Widget` を使用してモジュールを消費させることができます。 APIから実装の詳細を隠していて、プロトタイプを使用することによるパフォーマンスのメリットと圧倒的な優位性を得ることができます。

このウィジェットを使用するには、 `require()` を使用してウィジェットファイルを読み込んでインスタンス化し、css セレクタ文字列または dom 要素を使用して `.appendTo()` を呼び出します。

こんな感じ。

```javascript
var Widget = require('./widget.js');
var w = Widget();
w.appendTo('#container');
```

Widget はDOMに追加されます。

手続き的に HTML 要素を作成するのは、非常に単純なコンテンツでは問題ありませんが、それ以上には冗長で不明瞭です。 幸運にも、HTMLをJavaScriptモジュールに簡単にインポートするための変換が多数あります。

[brfs](https://npmjs.org/package/brfs) を使って Widget の例を拡張しましょう。 また、 `fs.readFileSync()` がHTML DOM要素に返す文字列を [domify](https://npmjs.org/package/domify) を使って有効にすることもできます。

```javascript
var fs = require('fs');
var domify = require('domify');

var html = fs.readFileSync(__dirname + '/widget.html', 'utf8');

module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = domify(html);
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
};
```

Widget は `widget.html` をロードしますので、作成しましょう。

```html
<div class="widget">
  <h1 class="name"></h1>
  <div class="msg"></div>
</div>
```

イベントを発行すると便利なことがよくあります。 組み込みイベントモジュールと継承モジュールを使用してイベントを発行する方法は次のとおりです。

```javascript
var fs = require('fs');
var domify = require('domify');
var inherits = require('inherits');
var EventEmitter = require('events').EventEmitter;

var html = fs.readFileSync(__dirname + '/widget.html', 'utf8');

inherits(Widget, EventEmitter);
module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = domify(html);
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
    this.emit('append', target);
};
```

今度は Widget のインスタンスで `'append'` イベントを聞くことができます。

```javascript
var Widget = require('./widget.js');
var w = Widget();
w.on('append', function (target) {
    console.log('appended to: ' + target.outerHTML);
});
w.appendTo('#container');
```
Wedget に要素を設定するためのメソッドを html に追加することができます。

```javascript
var fs = require('fs');
var domify = require('domify');
var inherits = require('inherits');
var EventEmitter = require('events').EventEmitter;

var html = fs.readFileSync(__dirname + '/widget.html', 'utf8');

inherits(Widget, EventEmitter);
module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = domify(html);
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
};

Widget.prototype.setName = function (name) {
    this.element.querySelector('.name').textContent = name;
}

Widget.prototype.setMessage = function (msg) {
    this.element.querySelector('.msg').textContent = msg;
}
```

要素の属性と内容の設定が冗長すぎる場合は、[hyperglue](https://npmjs.org/package/hyperglue) をチェックしてください。

最後に、 `widget.js` と `widget.html` を `node_modules/app-widget` に投げることができます。 Widget は brfs トランスフォームを使用しているので、 `package.json` を次のように作成できます。

```json
{
  "name": "app-widget",
  "version": "1.0.0",
  "private": true,
  "main": "widget.js",
  "browserify": {
    "transform": [ "brfs" ]
  },
  "dependencies": {
    "brfs": "^1.1.1",
    "inherits": "^2.0.1"
  }
}
```

そして今、アプリケーションのどこからでも require('app-widget') を呼び出すと、brfs は自動的に `widget.js` に適用されます！ Widget は独自の依存関係を維持することさえできます。 このようにして、他の Wedget へのカスケード変更を心配することなく、ある Wedget の依存関係を更新できます。

`.gitignore` に `node_modules/app-widget` の除外項目を追加してください。

```
node_modules/*
!node_modules/app-widget
```

ブラウザーといくつかのストリーミング HTML ライブラリーを使用して、node とブラウザー間でレンダリング・ロジックを共有する方法を知りたい場合は、[node とブラウザーで共用レンダリング](http://substack.net/shared_rendering_in_node_and_the_browser)の詳細を読むことができます。


## testing in node and the browser

モジュラーコードのテストはとても簡単です！ モジュール化の最大のメリットの1つは、インターフェイスが孤立してインスタンス化するのがはるかに簡単になり、自動化されたテストを簡単に実行できることです。

残念なことに、モジュールを使ってすぐにテストライブラリーを再生することはほとんどなく、明瞭なグローバル変数と無秩序なフロー制御を持つ独自のインタフェースをロールバックして、分離したクリーンなデザインになりがちです。

人々はまた、"mocking" について大騒ぎしますが、テストを念頭に置いてモジュールを設計する場合、通常は必要ありません。 IOをアルゴリズムとは別にして、モジュールのスコープを慎重に制限し、異なるインタフェースのコールバックパラメータを受け入れることで、コードをテストするのがずっと簡単になります。

たとえば、IOとプロトコルの両方を実行するライブラリがある場合は、[ストリーム](https://github.com/substack/stream-handbook) のようなインタフェースを使用して[IO層をプロトコルから分離する](https://www.youtube.com/watch?v=g5ewQEuXjsQ#t=12m30)ことを検討してください。

あなたが最初に想像していなかったさまざまな状況で、コードをテストして再利用しやすくなります。 これはテストの繰り返しテーマです。コードをテストするのが難しい場合は、モジュール化されていないか、抽象のバランスが間違っている可能性があります。 テストは後から行われるべきではありません。あなたの設計全体を知らせるべきであり、より良いインターフェースを書くのに役立ちます。


### testing libraries

#### [tape](https://npmjs.org/package/tape)

Tape は、node とブラウザの両方でうまく動作するように設計されています。 async インタフェースを持つ`index.js` があるとします。

```javascript
module.exports = function (x, cb) {
    setTimeout(function () {
        cb(x * 100);
    }, 1000);
};
```

このモジュールを [tape](https://npmjs.org/package/tape) でテストする方法は次のとおりです。 このファイルを `test/beep.js` に入れてみましょう。

```javascript
var test = require('tape');
var hundreder = require('../');

test('beep', function (t) {
    t.plan(1);
    
    hundreder(5, function (n) {
        t.equal(n, 500, '5*100 === 500');
    });
});
```

テストファイルは `test/` にあるので、 `require('../')` を実行することで親ディレクトリに `index.js` を要求できます。 `index.js` は、`main` フィールドを持つディレクトリに `package.json` がない場合に、node と browserify がモジュールを探すデフォルトの場所です。

`npm install tape` でインストールした後は、他のライブラリと同様に `require('tape')` できます。

文字列 `'beep'` は、テストのオプション名です。 `t.equal()` の第3引数は完全にオプションの記述です。

`t.plan(1)` は、1つのアサーションを期待していると言います。 アサーションが十分でないか、あまりにも多い場合は、テストは失敗します。 アサーションは `t.equal()` のような比較です。 テープには以下のアサーションプリミティブがあります。

- t.equal(a, b) - a と b を厳密に比較する ===
- t.deepEqual(a, b) - a と b を再帰的に比較する
- t.ok(x) - x が真でなければ失敗する

追加の説明引数はいつでも追加できます。

モジュールの実行は非常に簡単です！ node でモジュールを実行するには、`node test/beep.js` を実行します。

```bash
$ node test/beep.js
TAP version 13
# beep
ok 1 5*100 === 500

1..1
# tests 1
# pass  1

# ok
```

結果は標準出力に出力され、終了コードは `0` になります。

ブラウザでコードを実行するには、次のようにします。

```bash
$ browserify test/beep.js > bundle.js
```

`bundle.js` を `<script>` タグに入れます。

```html
<script src="bundle.js"></script>
```

そのHTMLをブラウザに読み込みます。 出力はデバッグコンソールに表示され、ブラウザに応じて F12、ctrl-shift-j、または ctrl-shift-k で開くことができます。

これは、ブラウザでテストを実行するのは少し面倒ですが、`testling` コマンドをインストールするとヘルプになることができます。 まずは…

```bash
npm install -g testling
```

それでは、単に `browserify test/beep.js | testling` を実行してください。

```bash
$ browserify test/beep.js | testling

TAP version 13
# beep
ok 1 5*100 === 500

1..1
# tests 1
# pass  1

# ok
```

`testling` はテストを実行するためにシステムにヘッドレスで実際のブラウザを起動します。

ここで `test/boop.js` ファイルを追加したいとします。

```javascript
var test = require('tape');
var hundreder = require('../');

test('fraction', function (t) {
    t.plan(1);

    hundreder(1/20, function (n) {
        t.equal(n, 5, '1/20th of 100');
    });
});

test('negative', function (t) {
    t.plan(1);

    hundreder(-3, function (n) {
        t.equal(n, -300, 'negative number');
    });
});
```

ここでは、2つの `test()` ブロックがあります。 2番目のテストブロックは、非同期であっても、最初のテストブロックが完全に終了するまで実行を開始しません。 `t.test()` を使用してテストブロックをネストすることさえできます。

`test/beep.js` を node と一緒に直接 `test/beep.js` と一緒に実行することもできますが、両方のテストを実行したい場合は、tape に付属しているコマンドランナーを最小限に抑えることができます。 tape コマンドを取得するには…

```bash
npm install -g tape
```

すぐに実行できます。

```bash
$ tape test/*.js
TAP version 13
# beep
ok 1 5*100 === 500
# fraction
ok 2 1/20th of 100
# negative
ok 3 negative number

1..3
# tests 3
# pass  3

# ok
```

`test/*.js` を browserify に渡して、ブラウザでテストを実行するだけです。

```bash
$ browserify test/* | testling

TAP version 13
# beep
ok 1 5*100 === 500
# fraction
ok 2 1/20th of 100
# negative
ok 3 negative number

1..3
# tests 3
# pass  3

# ok
```

これらすべてのステップをまとめると、`package.json` をテストスクリプトで設定できます。

```json
{
  "name": "hundreder",
  "version": "1.0.0",
  "main": "index.js",
  "devDependencies": {
    "tape": "^2.13.1",
    "testling": "^1.6.1"
  },
  "scripts": {
    "test": "tape test/*.js",
    "test-browser": "browserify test/*.js | testlingify"
  }
}`
```

これで、 `npm test` を実行して node でテストを実行し、 `npm run test-browser` を実行してブラウザでテストを実行できます。 npm を実行するときに `-g` を使用してコマンドをインストールすることについて心配する必要はありません。npm はプロジェクトにローカルにインストールされたすべてのパッケージの `$PATH` を自動的に設定します。

node でしか実行されないテストやブラウザでしか実行されないテストがある場合は、 `test/server` や `test/browser` などのサブディレクトリを `test/` にある両方の場所で実行するテストを行うことができます。 次に、関連するディレクトリを globs に追加するだけです。

```json
{
  "name": "hundreder",
  "version": "1.0.0",
  "main": "index.js",
  "devDependencies": {
    "tape": "^2.13.1",
    "testling": "^1.6.1"
  },
  "scripts": {
    "test": "tape test/*.js test/server/*.js",
    "test-browser": "browserify test/*.js test/browser/*.js | testling"
  }
}
```

一般的なテストに加えて、サーバー固有のテストとブラウザ固有のテストが実行されます。

より滑らかなものを望むなら、基本概念を取得したら [prova](https://www.npmjs.org/package/prova) をチェックしてください。


#### assert

コアアサーションモジュールは、簡単なテストを書くうえでも有効ですが、コールバックが正しく実行されるようにするのは難しい場合があります。

[macgyver](https://www.npmjs.org/package/macgyver) のようなツールでその問題を解決できますが、それは適切な DIY です。


### code coverage

#### coverify

browserify でコード範囲を調べる簡単な方法は、[coverify](https://npmjs.org/package/coverify) 変換を使用することです。

```bash
$ browserify -t coverify test/*.js | node | coverify
```

実際のブラウザでテストを実行するには…

```bash
$ browserify -t coverify test/*.js | testling | coverify
```

coverify は、各式が `__coverageWrap()` 関数でラップされるように各パッケージのソースを変換することによって動作します。

プログラム内の各式は一意のIDを取得し、 `__coverageWrap()` 関数は式が初めて実行されたときに `COVERED` `$FILE` `$ID` を出力します。

式が実行される前に、coverify は `COVERAGE` `$FILE` `$NODES` メッセージを出力して、式ノードを文字範囲としてファイル全体に記録します。

フル実行の出力は次のようになります。

```bash
$ browserify -t coverify test/whatever.js | node
COVERAGE "/home/substack/projects/defined/test/whatever.js" [[14,28],[14,28],[0,29],[41,56],[41,56],[30,57],[95,104],[95,105],[126,146],[126,146],[115,147],[160,194],[160,194],[152,195],[200,217],[200,218],[76,220],[59,221],[59,222]]
COVERED "/home/substack/projects/defined/test/whatever.js" 2
COVERED "/home/substack/projects/defined/test/whatever.js" 1
COVERED "/home/substack/projects/defined/test/whatever.js" 0
COVERAGE "/home/substack/projects/defined/index.js" [[48,49],[55,71],[51,71],[73,76],[92,104],[92,118],[127,139],[120,140],[172,195],[172,196],[0,204],[0,205]]
COVERED "/home/substack/projects/defined/index.js" 11
COVERED "/home/substack/projects/defined/index.js" 10
COVERED "/home/substack/projects/defined/test/whatever.js" 5
COVERED "/home/substack/projects/defined/test/whatever.js" 4
COVERED "/home/substack/projects/defined/test/whatever.js" 3
COVERED "/home/substack/projects/defined/test/whatever.js" 18
COVERED "/home/substack/projects/defined/test/whatever.js" 17
COVERED "/home/substack/projects/defined/test/whatever.js" 16
TAP version 13
# whatever
COVERED "/home/substack/projects/defined/test/whatever.js" 7
COVERED "/home/substack/projects/defined/test/whatever.js" 6
COVERED "/home/substack/projects/defined/test/whatever.js" 10
COVERED "/home/substack/projects/defined/test/whatever.js" 9
COVERED "/home/substack/projects/defined/test/whatever.js" 8
COVERED "/home/substack/projects/defined/test/whatever.js" 13
COVERED "/home/substack/projects/defined/test/whatever.js" 12
COVERED "/home/substack/projects/defined/test/whatever.js" 11
COVERED "/home/substack/projects/defined/index.js" 0
COVERED "/home/substack/projects/defined/index.js" 2
COVERED "/home/substack/projects/defined/index.js" 1
COVERED "/home/substack/projects/defined/index.js" 5
COVERED "/home/substack/projects/defined/index.js" 4
COVERED "/home/substack/projects/defined/index.js" 3
COVERED "/home/substack/projects/defined/index.js" 7
COVERED "/home/substack/projects/defined/index.js" 6
COVERED "/home/substack/projects/defined/test/whatever.js" 15
COVERED "/home/substack/projects/defined/test/whatever.js" 14
ok 1 should be equal

1..1
# tests 1
# pass  1

# ok
```

これらの COVERED 文と COVERAGE 文は単に標準出力に出力され、coverify コマンドに入力してよりきれいな出力を生成することができます。

```bash
$ browserify -t coverify test/whatever.js | node | coverify
TAP version 13
# whatever
ok 1 should be equal

1..1
# tests 1
# pass  1

# ok

# /home/substack/projects/defined/index.js: line 6, column 9-32

          console.log('whatever');
          ^^^^^^^^^^^^^^^^^^^^^^^^

# coverage: 30/31 (96.77 %)
```

プロジェクトにコードカバレッジを含めるには、 `package.json` の `scripts` フィールドにエントリを追加します。

```json
{
  "scripts": {
    "test": "tape test/*.js",
    "coverage": "browserify -t coverify test/*.js | node | coverify"
  }
}
```

ブラウジングとセットアップを簡素化する [covert](https://npmjs.com/package/covert) パッケージもあります。

```json
{
  "scripts": {
    "test": "tape test/*.js",
    "coverage": "covert test/*.js"
  }
}
```

coverdef または covert を devDependency としてインストールするには、`npm install -D coverify` または `npm install -D covert` を実行します。


## bundling

このセクションでは、バンドルについて詳しく説明します。

バンドルは、エントリファイルから始めて、依存関係グラフ内のすべてのソースファイルを歩き、単一の出力ファイルにパックするステップです。


### saving bytes

微調整したい最初のものの1つは、重複を避けるために npm がインストールするファイルをディスクに配置する方法です。

ディレクトリにクリーンインストールを実行すると、npm は通常、2つのモジュールが依存関係を共有する最上位のディレクトリに同様のバージョンを取り除きます。 しかし、より多くのパッケージをインストールすると、新しいパッケージは自動的に取り除かれません。 ただし、`npm dedupe` コマンドを使用して、`node_modules/` 内のパッケージの既存インストール済みセットをパッケージ化することができます。 重複の問題が残っている場合は、 `node_modules/` を削除して、最初からインストールすることもできます。

browserify には同じ正確なファイルが2回含まれませんが、互換性のあるバージョンはわずかに異なる場合があります。 browserify はバージョン対応でもなく、ノードが使用する `require()` アルゴリズムに従って、 `node_modules/` に配置されているものとまったく同じバージョンのパッケージを含みます。

`browserify --list` および `browserify --deps` コマンドを使用して、重複をスキャンするファイルが含まれているかどうかをさらに調べることができます。


### tinyify

[tinyify](https://github.com/browserify/tinyify) プラグインを使用して、適切なゼロ設定の最適化セットをバンドルに適用することができます。 出力サイズが大幅に削減されます。

```bash
$ browserify foo.js --plugin tinyify > bundle.js
```

tinyify には [browser-pack-flat](https://github.com/goto-bus-stop/browser-pack-flat) が含まれています。これは、デフォルトのブラウザパックと同じように Node モジュールの読み込み動作に従いません。 小さなバンドル出力にタイミングの問題がある場合は、 `-- no-flat` フラグを追加して、デフォルトの動作に戻すことができます。

```bash
$ browserify foo.js --plugin [ tinyify --no-flat ] > bundle.js
```

あらゆる種類の他の最適化が引き続き適用されるため、バンドルサイズの大幅な増加が見られるはずです。


### standalone

node で動作する `--standalone`、グローバルを持つブラウザ、AMD 環境で UMD バンドルを生成できます。

バンドルコマンドに `--standalone NAME` を追加するだけです。

```bash
$ browserify foo.js --standalone xyz > bundle.js
```

このコマンドは、 `foo.js` の内容を外部モジュール名 `xyz` でエクスポートします。 モジュールシステムがホスト環境で検出された場合、それが使用されます。 それ以外の場合は、`xyz` という名前の window グローバルがエクスポートされます。

ドット構文を使用して名前空間階層を指定することができます。

```bash
$ browserify foo.js --standalone foo.bar.baz > bundle.js
```

window グローバルモードのホスト環境にすでに `foo` または `foo.bar` がある場合、browserif yはそれらのオブジェクトにそのエクスポートを添付します。 AMD と `module.exports` モジュールは同じように動作します。

ただし、スタンドアロンは単一のエントリまたは直接必要なファイルでのみ動作します。

### external bundles
### ignoring and excluding

browserify 用語では、 `"ignore"` は、モジュールの定義を空のオブジェクトで置き換えることを意味します。 `"exclude"` とは、モジュールを依存関係グラフから完全に削除することです。

無視したり除外したりするのと同じ目標の多くを達成するもう1つの方法は、 `package.json` の `"browser"` フィールドです。このフィールドは、このドキュメントの別の部分で説明しています。


#### ignoring

無視は、一部のコードパスでのみ使用されるノード固有モジュールの空の定義をスタブするように設計された楽観的な戦略です。 例えば、あるモジュールが、node でしか動作しないコードの特定のチャンクに対して動作するライブラリを必要とする場合…

```javascript
var fs = require('fs');
var path = require('path');
var mkdirp = require('mkdirp');

exports.convert = convert;
function convert (src) {
    return src.replace(/beep/g, 'boop');
}

exports.write = function (src, dst, cb) {
    fs.readFile(src, function (err, src) {
        if (err) return cb(err);
        mkdirp(path.dirname(dst), function (err) {
            if (err) return cb(err);
            var out = convert(src);
            fs.writeFile(dst, out, cb);
        });
    });
};
```

browserify は既に空のオブジェクトを返すことで `'fs'` モジュールを 「無視」しますが、静的解析トランスフォームやランタイム・ストレージ fs 抽象化のような特別なステップを必要とせずにブラウザの `.write()` 関数は機能しません。

しかし、実際に `convert()` 関数が必要だが、最終バンドルで `mkdirp` を見たくない場合は、`b.ignore('mkdirp')` または `browserify --ignore mkdirp` を使って `mkdirp` を無視することができます。 `require('mkdirp')` は例外をスローしないため、空のオブジェクトを返すだけなので、コードはブラウザで `write()` を呼び出さなければ動作します。

一般的に言えば、主にアルゴリズム（パーザ、フォーマッタ）がIOを自分自身で行うモジュールは良い考えではありませんが、これらのトリックではブラウザのモジュールをブラウザで使用できるようになります。

コマンドラインで `foo` を無視するには、次のようにします。

```bash
browserify --ignore foo
```

いくつかのバンドルインスタンス `b` で api から `foo` を無視するには…

```bash
b.ignore('foo')
```


#### excluding

もう一つの関連することは、出力からモジュールを完全に削除し、実行時に `require('modulename')` が失敗することです。 これは、前もって定義された `require()` 定義にカスケードするように複数のバンドルに分割する場合に便利です。

たとえば、プライマリバンドルに表示したくない jquery のバンドルされたスタンドアロンバンドルがある場合は、次のようになります。

```bash
$ npm install jquery
$ browserify -r jquery --standalone jquery > jquery-bundle.js
```

`main.js` で `require('jquery')` するだけです。

```javascript
var $ = require('jquery');
$(window).click(function () { document.body.bgColor = 'red' });
```

jquery distバンドル先に配置にして、次のように書くことができます。

```html
<script src="jquery-bundle.js"></script>
<script src="bundle.js"></script>
```

jquery の定義が　`bundle.js` に表示されていない場合、 `main.js` をコンパイルするときに、jquery を除外することができます。

```bash
browserify main.js --exclude jquery > bundle.js
```

コマンドラインで `foo` を除外するには…

```bash
browserify --exclude foo
```

いくつかのバンドルインスタンス `b` を持つ api から `foo` を除外するには…

```javascript
b.exclude('foo')
```

### browserify cdn


## shimming

残念なことに、パッケージの中には、Node 形式の commonjs エクスポートで書かれていないパッケージもあります。グローバルまたは AMD で機能をエクスポートするモジュールには、これらの厄介なパッケージをブラウザが理解できるものに自動的に変換するのに役立つパッケージがあります。


### browserify-shim

非 commonjs パッケージを自動的に変換する1つの方法は、 [browserify-shim](https://npmjs.org/package/browserify-shim) です。

[browserify-shim](https://npmjs.org/package/browserify-shim) は変換としてロードされ、`package.json` から `"browserify-shim"` フィールドも読み込まれます。

私たちが `./vendor/foo.js` に配置した面倒なサードパーティのライブラリを使用して、その機能を `FOO` というwindow グローバルとしてエクスポートする必要があるとします。 `package.json` を次のように設定することができます。

```json
{
  "browserify": {
    "transform": "browserify-shim"
  },
  "browserify-shim": {
    "./vendor/foo.js": "FOO"
  }
}
```

`require('./vendor.foo.js')` すると、 `./vendor/foo.js` がグローバルなスコープに入れようとした `FOO` 変数が得られましたが、その試みはグローバルな汚染を防ぐために隔離されたコンテキストにシムを切ってしまいました。

`./vendor/foo.js` をロードするために常に相対パスを使用する必要はなく、ブラウザフィールドを使用して `require('foo')` を動作させることさえできます。

```json
{
  "browser": {
    "foo": "./vendor/foo.js"
  },
  "browserify": {
    "transform": "browserify-shim"
  },
  "browserify-shim": {
    "foo": "FOO"
  }
}
```

今、`require('foo')` は `./vendor/foo.js` がグローバルスコープに配置しようとした `FOO` エクスポートを返します。


## partitioning

ほとんどの場合、1つまたは複数のエントリファイルが1つのバンドルされた出力ファイルにマップされるデフォルトのバンドル方法は、すべての javascript アセットを取得する単一の http リクエストまでの遅延を最小限に抑えることを考えると…

しかし、管理パネルなどのほとんどの訪問者がほとんどまたはまったく使用しないウェブサイトの部分では、この初期ペナルティが高すぎることがあります。 この分割は、[無視と除外のセクション](https://github.com/browserify/browserify-handbook#ignoring-and-excluding)で説明した手法で行うことができますが、手動で共有依存関係を抽出することは、大規模で流動的な依存グラフを退屈にする可能性があります。

幸運なことに、ブラウザ出力を別々のバンドルペイロードに自動的に因数分解できるプラグインがあります。


### factor-bundle

[factor-bundle](https://www.npmjs.org/package/factor-bundle) 分割は、エントリポイントに基づいて出力を複数のバンドルターゲットにブラウジングします。 各エントリーポイントに対して、エントリー固有の出力ファイルが構築されます。 2つ以上のエントリファイルが必要とするファイルは、共通のバンドルにまとめられます。

たとえば、2つのページ（ `/x` と `/y` ）があるとします。 各ページには、エントリポイント `/x` 用の `x.js` と `/y` 用の `y.js` があります。

`x.js` と `y.js` の両方で共有されている依存関係を含む `bundle/common.js` で、ページ固有のバンドル`bundle/x.js` と `bundle/y.js` を生成します。

```bash
browserify x.js y.js -p [ factor-bundle -o bundle/x.js -o bundle/y.js ] \
  -o bundle/common.js
```

これで各ページに2つのスクリプトタグを置くことができます。 `/x` は…

```html
<script src="/bundle/common.js"></script>
<script src="/bundle/x.js"></script>
```

ページ `/y` に次のように記述します。

```html
<script src="/bundle/common.js"></script>
<script src="/bundle/y.js"></script>
```

また、バンドルを ajax と非同期にロードすることも、スクリプトタグを動的にページに挿入することもできますが、ファクタバンドルは、バンドルをロードするのではなく、バンドルを生成することのみに関係します。

### partition-bundle

[partition-bundle](https://www.npmjs.org/package/partition-bundle) は、出力を分割バンドルのような複数のバンドルに分割しますが、特別な `loadjs()` 関数を使用して組み込みのローダーを含みます。

partition-bundle は、ソースファイルをバンドルファイルにマップする json ファイルを取ります。

```json
{
  "entry.js": ["./a"],
  "common.js": ["./b"],
  "common/extra.js": ["./e", "./d"]
}
```

次に、partition-bundle がプラグインとしてロードされ、マッピングファイル、出力ディレクトリ、および宛先 URL パス（動的ロードに必要）が渡されます。

```bash
browserify -p [ partition-bundle --map mapping.json \
  --output output/directory --url directory ]
```

すぐ追加することができます。

```html
<script src="entry.js"></script>
```

あなたのページにエントリファイルをロードする。 エントリファイルの内側から、`loadjs()` 関数を使って他のバンドルを動的にロードすることができます。

```javascript
a.addEventListener('click', function() {
  loadjs(['./e', './d'], function(e, d) {
    console.log(e, d);
  });
});
```


## compiler pipeline

バージョン5以降、browserify は、コンパイラパイプラインを[ラベル付きストリームスプライサ](https://www.npmjs.org/package/labeled-stream-splicer)として公開しています。

つまり、変換を内部パイプラインに直接追加または削除することができます。 このパイプラインは、ファイルの監視や複数のエントリポイントからのバンドルのファクタリングなどの高度なカスタマイズのためのきれいなインタフェースを提供します。

たとえば、「deps」がハッシュソースファイルに計算された後にパススルー変換を最初に挿入することで、組み込み整数ベースのラベリングメカニズムをハッシュIDで置き換えることができます。 次に、キャプチャしたハッシュを使用して独自のカスタムラベルを作成し、組み込みの「label」変換を置き換えることができます。

```javascript
var browserify = require('browserify');
var through = require('through2');
var shasum = require('shasum');

var b = browserify('./main.js');

var hashes = {};
var hasher = through.obj(function (row, enc, next) {
    hashes[row.id] = shasum(row.source);
    this.push(row);
    next();
});
b.pipeline.get('deps').push(hasher);

var labeler = through.obj(function (row, enc, next) {
    row.id = hashes[row.id];
    
    Object.keys(row.deps).forEach(function (key) {
        row.deps[key] = hashes[row.deps[key]];
    });
    
    this.push(row);
    next();
});
b.pipeline.get('label').splice(0, 1, labeler);

b.bundle().pipe(process.stdout);
```

出力フォーマットでIDの整数を取得する代わりに、ファイルハッシュを取得します。

```bash
$ node bundle.js
(function e(t,n,r){function s(o,u){if(!n[o]){if(!t[o]){var a=typeof require=="function"&&require;if(!u&&a)return a(o,!0);if(i)return i(o,!0);var f=new Error("Cannot find module '"+o+"'");throw f.code="MODULE_NOT_FOUND",f}var l=n[o]={exports:{}};t[o][0].call(l.exports,function(e){var n=t[o][1][e];return s(n?n:e)},l,l.exports,e,t,n,r)}return n[o].exports}var i=typeof require=="function"&&require;for(var o=0;o<r.length;o++)s(r[o]);return s})({"5f0a0e3a143f2356582f58a70f385f4bde44f04b":[function(require,module,exports){
var foo = require('./foo.js');
var bar = require('./bar.js');

console.log(foo(3) + bar(4));

},{"./bar.js":"cba5983117ae1d6699d85fc4d54eb589d758f12b","./foo.js":"736100869ec2e44f7cfcf0dc6554b055e117c53c"}],"cba5983117ae1d6699d85fc4d54eb589d758f12b":[function(require,module,exports){
module.exports = function (n) { return n * 100 };

},{}],"736100869ec2e44f7cfcf0dc6554b055e117c53c":[function(require,module,exports){
module.exports = function (n) { return n + 1 };

},{}]},{},["5f0a0e3a143f2356582f58a70f385f4bde44f04b"]);
```

組み込みのラベラーは外部の除外された構成をチェックするような他の処理を行います。そのため、それらの機能に依存すると、置き換えが困難になります。 この例は、コンパイラのパイプラインをハッキングすることでできる種類の例を示しています。


### build your own browserify
### labeled phases

browserify パイプラインの各フェーズには、フックできるラベルがあります。 適切なラベルに[ラベル付きストリーム・スプライサ](https://npmjs.org/package/labeled-stream-splicer) ハンドルを戻すには、`.get(name)` でラベルを取得します。 ハンドルを作成したら、パイプラインに `.push()`、`.pop()`、`.shift()`、`.unshift()`、および `.splice()` を実行して既存のトランスフォームストリームを削除できます。


#### recorder

レコーダーは、`deps` フェーズに送られた入力をキャプチャするために使用されます。そのため、後で `.bundle()` を呼び出すと再生されます。 以前のリリースとは異なり、v5 はバンドル出力を複数回生成できます。 これは、ファイルが変更されたときに再バンドルを監視するようなツールでは非常に便利です。


#### deps

`deps` フェーズでは、ファイルまたはオブジェクトを入力として `require()` し、依存関係グラフ内のすべてのファイルの json 出力ストリームを生成するために [module-deps](https://npmjs.org/package/module-deps) を呼び出します。

module-deps は以下のようなカスタマイズをして呼び出されます。

- package.json の browserify トランスフォームキーを設定する
- 外部ファイル、除外されたファイル、無視されるファイルのフィルタリング
- browserify コンストラクタの `opts.extensions` パラメータで設定された `.js` と `.json` とオプションのデフォルト拡張を設定する
- `process`、`Buffer`、`global`、`__dirname`、および `__filename` を検出および実装するグローバルな [insert-module-globals](https://github.com/substack/insert-module-globals) トランスフォームのコンフィグレーション
- browserify でシムになっている node の組み込みリストを設定する


#### json

この変換は `.json` 拡張子を持つファイルの前に `module.exports =` を追加します。


#### unbom

この変換により、バイトオーダーマーカーが削除されます。これは、Windowsのテキストエディターでファイルのエンディアンを示すために使用されることがあります。 これらのマーカーは node によって無視されるため、browserify は互換性のため無視します。


#### syntax

この変換では、構文エラー・パッケージを使用して[構文エラー](https://npmjs.org/package/syntax-error)をチェックし、行番号と列番号を含む構文上のエラーを通知します。


#### sort

この段階では、[deps-sort](https://www.npmjs.org/package/deps-sort) を使用してバンドルを確定的にするために書き込まれた行をソートします。


#### dedupe

この段階での変換では、ソート段階で [deps-sort](https://www.npmjs.org/package/deps-sort) によって提供された重複情報を使用して、内容が重複するファイルを削除します。


#### label

このフェーズでは、システムパス情報を公開し、バンドルサイズを整数ベースのIDに膨らませるファイルベースのIDを変換します。

`label` フェーズでは、 `opts.basedir` または `process.cwd()` に基づいてパス名を正規化して、システムパス情報が表示されないようにします。


#### emit-deps

このフェーズは、`label` フェーズの後の各行に対して`'dep'`イベントを発行します。


#### debug

`opts.debug` が `browserify()` コンストラクタに渡された場合、このフェーズは入力を変換して、`pack` フェーズで [browser-pack](https://npmjs.org/package/browser-pack) によって使用される `sourceRoot` および `sourceFile` プロパティを追加します。


#### pack

このフェーズでは、 `'id'` と `'source'` パラメータを持つ行を入力として変換し、[browser-pack](https://npmjs.org/package/browser-pack) を使用して連結された javascript バンドルを出力として生成します。


#### wrap

これは、最後の空の段階で、既存のメカニックに干渉することなく、カスタムのポスト変換を簡単に行うことができます。


### browser-unpack

[browser-unpack](https://npmjs.org/package/browser-unpack) は、コンパイルされたバンドルファイルを [module-deps](https://npmjs.org/package/module-deps) の出力と非常によく似た形式に変換します。

これは、既にコンパイルされたバンドルを調べたり変換したりする必要がある場合に非常に便利です。

例えば…

```bash
$ browserify src/main.js | browser-unpack
[
{"id":1,"source":"module.exports = function (n) { return n * 100 };","deps":{}}
,
{"id":2,"source":"module.exports = function (n) { return n + 1 };","deps":{}}
,
{"id":3,"source":"var foo = require('./foo.js');\nvar bar = require('./bar.js');\n\nconsole.log(foo(3) + bar(4));","deps":{"./bar.js":1,"./foo.js":2},"entry":true}
]
```

この分解は、[factor-bundle](https://www.npmjs.org/package/factor-bundle) や [bundle-collapser](https://www.npmjs.org/package/bundle-collapser) などのツールで必要となります。



## plugins

ロードされると、プラグインは browserify インスタンス自体にアクセスできます。


### using plugins

プラグインは、必要な機能を実行するのに変換またはグローバル変換が十分強力でない場合に限り、控えめに使用する必要があります。

コマンドラインでプラグインを `-p` でロードすることができます。

```bash
$ browserify main.js -p foo > bundle.js
```

`foo` というプラグインをロードします。 `foo` は `require()` で解決されるので、ローカルファイルをプラグインとしてロードするにはパスを `./` で始まり、プラグインを `node_modules/foo` からロードするには `-p foo` を実行します。

最初の引数としてプラグイン名を含めて、プラグイン式全体を角括弧で囲んでプラグインにオプションを渡すことができます。

```bash
$ browserify one.js two.js \
  -p [ factor-bundle -o bundle/one.js -o bundle/two.js ] \
  > common.js
```

このコマンドライン構文は、[subarg](https://npmjs.org/package/subarg) パッケージによって解析されます。

browserify プラグインのリストを表示するには、キーワード "browserify-plugin" を持つパッケージの npm を参照してください：[http://npmjs.org/browse/keyword/browserify-plugin](http://npmjs.org/browse/keyword/browserify-plugin)


### authoring plugins

プラグインを作成するには、バンドルインスタンスとオプションのオブジェクトを引数として受け取る単一の関数をエクスポートするパッケージを作成します。

```javascript
// example plugin

module.exports = function (b, opts) {
  // ...
}
```

プラグインは、イベントをリッスンするか、トランスフォームをパイプラインにスプライスすることによって、バンドルインスタンス `b` で直接動作します。 非常に良い理由がない限り、プラグインはバンドルメソッドを上書きしてはいけません。

