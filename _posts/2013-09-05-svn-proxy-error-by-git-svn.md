---
layout: post
title: "Subversionのproxy設定でいつもハマるやつ"
description: ""
category: 
tags: [Git, Subversion]
---

## あらすじ

`git svn` しようとしたらエラー。

{% highlight console %}
$ git svn rebase
Malformed file: /c/Users/xxx/.subversion/servers:68: Option expected at
/usr/lib/perl5/site_perl/Git/SVN/Ra.pm line 81
{% endhighlight %}

## 環境

- Windows

svn バージョン等は失念。まあ、多分バージョンはあまり関係ないと思われる？

## 結論

`git-svn` の問題ではなく proxy 環境下 においての svn 設定ミスだった。

## 原因

`.subversion/server` の該当部分を見に行くとこうなっている。

{% highlight console %}
[global]
# http-proxy-exceptions = *.exception.com, www.internal-site.org
 http-proxy-host = proxy.xxx.jp
 http-proxy-port = 8080
{% endhighlight %}

proxy 設定を追加するためにコメントアウトを消したが。

- コメントアウト `#` を **一文字** 消しただけではダメ(上記の状態)
- **スペース** も消さなければならない

ただしくはこう。

{% highlight console %}
[global]
# http-proxy-exceptions = *.exception.com, www.internal-site.org
http-proxy-host = proxy.xxx.jp
http-proxy-port = 8080
{% endhighlight %}

これ、毎回設定する時にひっかかってるような気がする。
