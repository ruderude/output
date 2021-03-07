[トップページ](../index.md)  

# モジュールについて

## モジュールの基礎

しまぶーのIT大学のモダンJavaScript講座（モジュール編）を観てまとめてみました  
[モダンJavaScript講座](https://youtube.com/playlist?list=PLwM1-TnN_NN4SV6DEs4OtfA51Up6XzTfB)

### 今回学ぶモジュールの形式

- CommonJS
- ES modules

CommonJSはサーバーサイドで使用する形式。
今回はブラウザのES modulesを使うやり方を紹介していきます。

※ここに少し詳しく書いてあります  
[JSモジュール化するしくみの規格はCommonJS、AMD、UMD、ES6(ES2015)の4つ](http://ytyaru.hatenablog.com/entry/2019/03/29/000000)

### 名前空間（スコープ）の問題

#### モジュールがない場合、どのような問題が生じていたのか

`<script>`タグのスコープの範囲
↓これは通る。上のスコープで宣言した変数が下のスコープでも使用できる。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>タイトル</title>
</head>
<body>
    <div id="main"></div>

    <script>
        const name = 'くんしー!!!';
    </script>

    <script>
        document.getElementById("main").innerText = name; // くんしー!!!
    </script>
</body>
</html>
```

↓これはエラー（constは再代入不可）

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>タイトル</title>
</head>
<body>
    <div id="main"></div>

    <script>
        const name = 'くんしー!!!';
    </script>

    <script>
        const name = '山ちゃん'; //上のスコープが効いているため再宣言はできない
        document.getElementById("main").innerText = name; // Uncaught SyntaxError: Identifier 'name' has already been declared
    </script>
</body>
</html>
```

つまり`<script>`タグを分けても同一スコープである。
さらに読み込む外部ファイルが増えていったら大変なことに・・・

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>タイトル</title>
</head>
<body>
    <div id="main"></div>

    <script src="./file1.js"></script>
    <script src="./file2.js"></script>
    <script src="./file3.js"></script>
    <script src="./file4.js"></script>
    <script src="./file5.js"></script>
    <script src="./file6.js"></script>
    .
    .
    .
</body>
</html>
```

こういった理由で、今のjQueryなどを使ったWeb制作の開発も将来的には少なくなる。
モジュールを使った開発に置き換わって行く。

### モジュールを使ったコードに変換

`<script>`内にtype="module"と入れてやるとスコープがわかれる。

```html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>タイトル</title>
</head>

<body>
    <div id="main"></div>
    <div id="sub"></div>

    <script type="module">
        const name = 'くんしー!!!'; //上と下のスコープは別モノ!!
    </script>

    <script type="module">
        const name = '山ちゃん';
        document.getElementById("main").innerText = name; // 山ちゃん
    </script>

    <script type="module">
        const name = 'シドちゃん';
        document.getElementById("sub").innerText = name; // シドちゃん
    </script>
</body>

</html>
```

### モジュールの遅延評価について

モジュール化するとheadダグ内に置いても問題なく動きます。
これは遅延評価といってファイルの上から順番に評価されるものが、モジュール化した結果、ページの最後に読み込まれるようになったためです。

```html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>タイトル</title>
    <script type="module">
        const name = 'くんしー!!!'; // headタグ内でもOK！
    </script>

    <script type="module">
        const name = '山ちゃん';
        document.getElementById("main").innerText = name; // 山ちゃん
    </script>

    <script type="module">
        const name = 'シドちゃん';
        document.getElementById("sub").innerText = name; // シドちゃん
    </script>

    <!-- 外部ファイルもモジュール化OK -->
    <script type="module" src="./file1.js"></script>

</head>

<body>
    <div id="main"></div>
    <div id="sub"></div>
</body>

</html>
```

<br>

### モジュール間でのデータのやりとり、exportとimport

よく使う方から、、、

#### 名前付きexport,import（Named import / export）の書き方をマスターしよう！

main.jsとsub.jsを外部にきって。。。

```html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>タイトル</title>
</head>

<body>
    <div id="main"></div>

    <!-- 読み込みはmain.jsのみ -->
    <script type="module" src="./main.js"></script>

</body>

</html>
```

sub.jsでexportする

```js
// sub.js

export const name = "訓志"; // exportして
```

main.jsでimportする

```js
// main.js

import { name } from "./sub.js"
document.getElementById("main").innerText = name; // importすると、訓志が受けとれて使用できる

```

これは関数やクラスなどにも使えます

```js

export const function hogehoge(){
    // 関数をexport
};

export class MyClass {
    // クラスをexport
}
```

モジュールの最後にエキスポートしたい全てを羅列できる。

```js

const name = ...;
function myFunc() {
  ...
}

export { name, myFunc };
```

別の名前でもエキスポートできる。

```js

const name = ...;
function myFunc() {
  ...
}

export { name as myName, myFunc as theFunc };
```

importにも色々なやり方があり混乱する・・・w

```js
// main.js

import { name } from "./sub.js"

import { name1, name2 } from 'src/mylib'; //複数import

import { name1 as myName1, name2 } from 'src/mylib'; //別名を付けてimport

```

<br>

### Default import/exportの書き方をマスターしよう！

動画内でもいきなり「こちらは使うべきではない(Default import/export)」と言っています。
「デフォルト」って書かれてると、こちらを優先的に使いたくなりますよね・・・w

主なデメリット

- **エディタ上で補完が効きにくくなる**
- **import/export時のタイポに気づきにくい**
- **リファクタリング時に名前が違うことによって検出しにくい**

#### export側

```js

export default const name = "訓志"; // エラー！これはできません。変数はカンマ区切りで複数定義できるため直接のexport defaultはできません。

export default "訓志"; // まず書かないが値を直接exportすることは可能。

export default const function hogehoge(){
    // 関数をexport default。宣言と同時にexport defaultする場合は識別子のない無名関数でもOK
};

export default class MyClass {
    // クラスをexport default。宣言と同時にexport defaultする場合は識別子のない無名クラスでもOK
}

export default name; //変数をexport defaultする場合はこの書き方。

// これは例を書いています。export default は１ファイルに1回しか使えません!!
```

↓こんな感じで書くこともできる。

```js

const name = "訓志"; // 変数をexport default

export const function hogehoge(){ ... }; // これは名前付きexport

export class MyClass { ... } // これも名前付きexport

export default name; // これと下のコードは同じ

export default "訓志"; // export defaultは評価された値をexportするので、上のコードはこのコードと同じ意味になります。つまり名前を見ていません。
```

#### import側

上のexportを受けとる。

```js
import name from './sub.js'; // export defaultをimport

import anotherName from './sub.js'; // export default側は評価された値をexportしているので勝手な名前を付けられる。

import name, { hogehoge, MyClass } from './sub.js'; // 名前付きexportと一緒にimportの場合

```

<br>

### モジュールの特殊な Import / Export について学ぼう


#### all importについて

まとめてimportして、ドット記法でアクセスできる。


```js
// sub.js

const kunshi = "訓志";
const sid = "シド";

export { kunshi, sid };

```

```js
// main.js

import * as allName from './sub.js';

console.log( allName.kunshi ); // 訓志
console.log( allName.sid ); // シド

```

#### Re-Exportについてはまた今度。。。

### まとめ

よく使うタイプを決めて、不都合な時は都度調べて実装していこう

- **基本的にはES modulesの仕様で「名前付きexport/import」を使う。**
- **エイリアス（別名）はそのファイル内で使いやすいようにimportで行う。**

[トップページ](../index.md)  


