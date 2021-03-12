---
layout: default
title: トップページですよ
---

# マークダウンによるアウトプットメモ

こんにちは。
開発チームのkunshiです。
アウトプットを記録。
目次より下はマークダウンの書き方の例が置いてあるだけです。

##　もくじ

[webpackについて](./JavaScript/webpack.md)  
[Laravelでログイン後、直前のページに遷移したい](./laravel/auth_redirect.md)  
[モジュールについて](./JavaScript/module.md)  
[議事録の書き方](./business/Minutes.md)  
[テーブル結合(inner joinとouter join)](./DB/join1.md)  


## Markdown という文書フォーマットをご存知の方は多数いると思いますし、常用されている方も多いと思います

- Markdown でリストを書くのは簡単です
- 例えば、こんな風に、行頭に「-」と半角スペースを入力するだけ
  - リストのネストも行頭のスペースやタブを入れるだけでできます
- とても簡単ですね

1. 文章構造が簡単なため、他のフォーマットに変換することが容易
1. 文書を簡単な記号とルールで簡素化できるため、ドキュメントの推敲にフォーカスできる
1. Markdown 文書内にコードをソースコードを埋め込むことができる（実行できるわけではない）。特に技術方面では、文章とそれを補完するコードのパートという構成を作りやすい
1. 単なる文書フォーマットのため、特定のエディタに依存することなく文章が書ける

[webpackについて](./JavaScript/webpack.md)
[Laravelでログイン後、直前のページに遷移したい](./laravel/auth_redirect.md)  

```php
<?php
 
class hogehoge
{
    function text($message)
    {
        echo $message;
    }
}
 
$a = new hogehoge();
$a->text();
```

Column A | Column B | Column C
---------|----------|---------
 A1 | B1 | C1
 A2 | B2 | C2
 A3 | B3 | C3

 Markdown では、このように __強調__ します。

- Markdown でリストを書くのは簡単です
- 例えば、こんな風に、行頭に「-」と半角スペースを入力するだけ
  - リストのネストも行頭のスペースやタブを入れるだけでできます
- とても簡単ですね

>引用したい文章<br>
>引用したい文章<br>
>引用したい文章<br>
>引用したい文章<br>

引用したい文章
