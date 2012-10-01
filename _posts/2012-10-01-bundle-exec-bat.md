---
layout: post
title: "bundle execを省略したいのでバッチを作った(Windows版)"
description: ""
category: 
tags: [Ruby, Bundle, Windows]
---
{% include JB/setup %}

## あらすじ

Bundler、非常によいツールなんだけど、bundle exec がめんどい。シェルで実行する場合はbundle-exec.shというツールを使う事でうまい事回避してくれるんだけど、Windowsでもこれを実現したい。 - [bundle exec を省略する - おもしろWEBサービス開発日記](http://d.hatena.ne.jp/willnet/20110612/1307849053)

シェルを参考にバッチファイルを作成して同じ事できないかな。

## 参考サイト

- [コマンドのエイリアス設定(DOSKEY)](http://www.adminweb.jp/command/command/index2.html)
- [whichコマンドを作る ](http://www.atmarkit.co.jp/fwin2k/win2ktips/319which/which.html)
- [コマンドプロンプトを使ってみよう！　－バッチファイル－](http://ykr414.com/dos/dos05.html#07)
- [Windows 2000 コマンドライン徹底活用](http://www.atmarkit.co.jp/fwin2k/operation/command008/command02.html)
- [コマンド別/for](http://otnx.jp/CMD/%A5%B3%A5%DE%A5%F3%A5%C9%CA%CC/for/)
- [UNIXライクなパス操作をwindowsのcmd.exeのバッチファイルのみで実現](http://pgkiss.web.fc2.com/windows/batch-file.html)
- [Windowsのコマンドプロンプトは変態（一部）](http://d.hatena.ne.jp/superstring04/20080301/1204373672)

## 調査

ソースを読む限り、やっている処理は

1. `ruby` とか` rspec` とかbundle exec hogehogeしたいコマンドにaliasをはる(rubyならbundle-exec.sh rubyとなるように)
1. `bundle` コマンドが使えるのか探してみる。使えない場合はそのままコマンドを実行する
1.  `bundle` が使える場合、今いるパスに `Gemfile` ファイルがあるのか調べる。ない場合は一つ階層を上がってまた `Gemfile` があるか探す
1. 上記をを繰り返す。 `Gemfile` がない場合はそのままコマンドを実行する
1. `Gemfile` があった場合、 `bundle exec` を付加してコマンドを実行する

という感じになっている。

## ソース

[gosyujin/bundler-exec-bat](https://github.com/gosyujin/bundler-exec-bat)

## 手順

### aliasをはる

コマンドプロンプトの場合は、マクロ定義ファイルを作成して、それを起動時に読むようにバッチファイルを作る、起動オプションを変更する必要があるらしい。(またはレジストリに登録など)

定義ファイルは以下のように記述する。とりあえずruby, rspec, herokuを使えるように。必要なコマンドは適宜このファイルに追加する事になる。

    heroku=bundle-exec heroku $*
    rspec=bundle-exec rspec $*
    ruby=bundle-exec ruby $*

定義ファイルを読み込む場合は

`> doskey /macrofile=MACRO_FILE`

alias(マクロ)を確認するには

`> doskey /macros`

### それ以降

それ以降の処理はバッチファイルに記述する。

バッチファイルをがっつり書く機会もあまりなかったので、結構試行錯誤。(pwdをWindowsで実現させるには？　コマンドの実行結果を変数に格納するには？　while文ないの？　等々)

とりあえず、動く版を作る事ができた。一応、Documents and Settings下でも実行できたのでパスにスペースが入っていても大丈夫そう。

これをパスが通っているところに放り込み、下記のようなfizzbuzzディレクトリで実行してみる。

    C:\fizzbuzz
      ┣.bundle
      ┣Gemfile
      ┣Gemfile.lock
      ┣fizzbuzz.rb
      ┣autotest/
      ┣spec/
      ┗vendor/
        ┗bundle
          ┗ruby
           ┗1.9.1
             ┗…

まずは普通に。

{% highlight console %}
$ ruby fizzbuzz.rb 3
bundler is found: bundle exec ruby fizzbuzz.rb 3
Fizz
{% endhighlight %}

ちょっと下の階層から実行しても、無事にGemfileを探してbundle exec。

{% highlight console %}
$ cd autotest\
$ ruby ../fizzbuzz.rb 5
bundler is found: bundle exec ruby ../fizzbuzz.rb 5
Buzz
{% endhighlight %}

Gemfileを消して `Ruby` コマンド実行。

{% highlight console %}
$ ls
autotest/    fizzbuzz.rb* spec/
$ ruby fizzbuzz.rb 15
bundler is NOT found or Gemfile is NOT found: ruby fizzbuzz.rb 15
FizzBuzz
{% endhighlight %}

ちゃんと `bundle exec` なしで実行できた。

## 残件

- なんか重い

結構力技でやってるから…。 