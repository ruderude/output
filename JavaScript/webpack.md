# webpackについて

## webpackとは

- モジュールバンドラーである
  - 複数のモジュールの依存関係を解決してひとつにまとめるもの（バンドリング）

クライアントとサーバは「HTTP/1.1」というプロトコルで通信されていてリクエスト回数によってパフォーマンスが落ちるため、ファイルをまとめて送るために使われるのがwebpackである。

***

## Webシステム開発者の願い

>開発する時は、なるべく機能を分けたい
>実行する時は、なるべく機能をまとめたい

## なぜ使うのか？

<font color="red">大きな利点は2つ</font>

- **開発効率アップ**
  - 機能ごとにファイルを分けて開発をすることによって、コードが整理されて開発効率、保守性が上がる。
- **システムの性能アップ**
  - たくさんのjsファイルを１ページで読み込もうとするとリクエストがその数だけ発生し、時間がかかってしまうが、１つのファイルにまとめることで、リクエストも１回だけにし性能を高めることができます。

## JavaScriptのモジュール機能を使う

node.js、npmを使用してJavaScriptファイルやCSSファイルをまとめる。

CSSを結合するには**style-loader、css-loader**が必要。

モジュールを読み込む方法としてrequireとかimportがあるが、その違いについてはこちらの記事で解説している。
[jsのimportとrequireの違い](https://qiita.com/minato-naka/items/39ecc285d1e37226a283)

### バンドル対象ファイル

この2つのファイルをまとめたいとする。

```js
//sub1.js
module.exports.sub1text = function() {
        console.log("sub1.jsです。");
}
```

```js
//sub2.js
module.exports.sub2text = function() {
        console.log("sub2.jsです。");
}
```

### エントリーポイントファイル

２つのファイルをまとめて1つのファイルに。

```js
//index.js

//まとめたいファイルをインポート
var sub1 = require(./sub1.js);
var sub2 = require(./sub2.js);

//インポートしたファイルの関数を実行できる
sub1.sub1text();
sub2.sub2text();
```

### 設定ファイル

webpack.config.jsはwebpackの設定ファイルです。

```js
//webpack.config.js

module.exports = {
        entry: "./index.js",
        output: {
                path: "./",
                filename: "main.js"
        }
}
```

entryにエントリーポイントファイルを書いて、outputに、バンドルされたファイル名と出力先パスを書きます。
あとは出力されたjsファイルをHTMLから呼び出しましょう。

## ちょっと小難しい「webpack.config.js」

module.exports内の記述で「test」というプロパティがプロパティ名からは想像がつきにくいが「ルールに該当するファイル」を表します。

```js
//webpack.config.js

const path = require('path');

module.exports = {
    entry: path.join(__dirname, 'src', 'index.js'),

    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'main.js'
    },

    module: {
        rules: [
            {
                test: /\.css$/,
                use:['style-loader','css-loader']
            }
        ]
    }
};
```

ちなみに上のコードは下のコードでも通ります。
当たり前なのですが、JavaScript構文の理解がふわふわだと設定ファイルが読めずに困惑しますよね

```js
//webpack.config.js

const path = require('path');

module.exports = {
    entry: path.join(__dirname, 'src', 'index.js'),

    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'main.js'
    },

    module: {
        rules: [
            {
                test: '',
                use:['','']
            }
        ]
    }
};

module.exports.module.rules[0].test = /\.css$/;
module.exports.module.rules[0].use[0] = 'style-loader';
module.exports.module.rules[0].use[1] = 'css-loader';
```

**webpack.config.jsもつまりはnode.jsでありJavaScriptで書かれている。**

### 参考資料

わかりにくいwebpackをわかりやすく解説！ <br>
[webpack.config.jsをスラスラ読む方法](https://youtu.be/gdQx7bnQCrs)