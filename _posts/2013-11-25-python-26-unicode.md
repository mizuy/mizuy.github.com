---
layout: post
title: "Python 2.6 Unicode化まとめ"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

[gist](https://gist.github.com/mizuy/961545)の移植テスト。

  It is strongly recommended that you do use the Unicode type to represent strings, since it will raise a warning if a non-unicode Python string is passed from the user application. http://www.sqlalchemy.org/docs/dialects/sqlite.html

といわれたので、内部文字コードをunicodeにしたのでそのメモ。

## Python 2系列での文字列を3種類に分けて考える。

注: encodingはutf-8のみを考える。

- ''unicode'': typeがunicode なもの。u'hogehoge', u'ほげほげ' など。
- ''ascii-string'': typeがstr でかつ、ASCII文字のみを含むもの。'hogehoge' など。
- ''encoded-string'': typeがstr でかつ、ASCII文字以外を含む物。'ほげほげ' など。

### unicode/ascii-string/encoded-string間の変換

unicode から encoded-string への変換::

{% highlight python %}
{% raw %}
  >>> u'ほげほげ'
  u'\u307b\u3052\u307b\u3052'
  >>> u'ほげほげ'.encode('utf-8')
  '\xe3\x81\xbb\xe3\x81\x92\xe3\x81\xbb\xe3\x81\x92'
  >>> print u'ほげほげ'.encode('utf-8')
  ほげほげ
{% endraw %}
{% endhighlight %}

encoded-string から unicode への変換::

{% highlight python %}
{% raw %}
   >>> 'ほげほげ'.decode('utf-8')
   u'\u307b\u3052\u307b\u3052'
   >>> unicode('ほげほげ','utf-8')
   u'\u307b\u3052\u307b\u3052'
{% endraw %}
{% endhighlight %}

ascii-stringとunicodeが合体すると、unicodeになる::

{% highlight python %}
{% raw %}
   >>> 'hogehoge'+u'ほげほげ'
   u'hogehoge\u307b\u3052\u307b\u3052'
   >>> '-'.join(c for c in u'ほげほげ')
   u'\u307b-\u3052-\u307b-\u3052'
{% endraw %}
{% endhighlight %}

ascii-stringとencoded-stringが合体すると、encoded-stringになる::

{% highlight python %}
{% raw %}
   >>> 'hogehoge'+'ほげほげ'
   'hogehoge\xe3\x81\xbb\xe3\x81\x92\xe3\x81\xbb\xe3\x81\x92'
   >>> '-'.join(c for c in 'ほげほげ')
   '\xe3-\x81-\xbb-\xe3-\x81-\x92-\xe3-\x81-\xbb-\xe3-\x81-\x92'
{% endraw %}
{% endhighlight %}

encoded-stringとunicodeを合体させようとすると、エラーになる。::

{% highlight python %}
{% raw %}
   >>> 'ほげほげ'+u'ほげほげ'
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   UnicodeDecodeError: 'ascii' codec can't decode byte 0xe3 in position 0: ordinal not in range(128)
{% endraw %}
{% endhighlight %}

### 健全な変換関数

ascii-stringを変換する関数は、素直に実装されていれば、encoded-string/unicodeも変換できる。::

{% highlight python %}
{% raw %}
  >>> def convert(input):
  ...     return input.replace('<blockquote>','<pre_>').replace('</blockquote>','</pre_>')
  ... 
  >>> convert('<blockquote>ほげほげ</blockquote>')
  '<pre_>\xe3\x81\xbb\xe3\x81\x92\xe3\x81\xbb\xe3\x81\x92</pre_>'
  >>> convert(u'<blockquote>ほげほげ</blockquote>')
  u'<pre_>\u307b\u3052\u307b\u3052</pre_>'
{% endraw %}
{% endhighlight %}

健全な変換関数とは

- encoded-stringの入力に対してはencoded-string or ascii-stringを返す
- unicodeの入力に対してはunicode or ascii-stringを返す

実装法は

- encoded-stringやunicodeを使わない。入力が汚染される。
- cStringIO.StringIOはunicodeを扱えないので使わない。StringIO.StringIOをつかう。http://docs.python.org/library/stringio.html
- regexを使うときは、re.Uオプションをつける

convert内でencoded-string 'ほ'が存在するため、unicodeを受け付けると死亡::

{% highlight python %}
  >>> def convert(input):
  ...     return input.replace('ほ','HO')
  ... 
  >>> convert('ほげー')
  'HO\xe3\x81\x92\xe3\x83\xbc'
  >>> convert(u'ほげー')
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 2, in convert
  UnicodeDecodeError: 'ascii' codec can't decode byte 0xe3 in position 0: ordinal not in range(128)
{% endhighlight %}

ここではこうした関数を、encoded-stringに汚染した関数、と呼ぶことにする。
同様に、convert内でunicodeを使うと、unicodeに汚染した関数ができ、encoded-stringを受け付けると死亡。

こうした点に注意して健全に実装すれば複雑な関数でも自動的にencoded-string/unicodeのどちらも扱える

{% highlight python %}
{% raw %}
  # coding:utf-8
  from StringIO import StringIO
  from mizwiki.wiki2html import *
  from nose.tools import *
  import re

  wiki = u"""
  書いた文章とその行間隔は見た目そのまま出力されます。

  文章と文章の間に空行を挟むと、それぞれがパラグラフ(p)となります。
  空行をはさまずに次の文を書くと、パラグラフ(p)内で改行(br)されます。

  チルダを使うと強制的に改行されます。
  これはグラフ要素などで使用することを意図しています。~~~~通常の文では使う必要はありません。
  """
  result = u"""<p>書いた文章とその行間隔は見た目そのまま出力されます。</p>
  <p>文章と文章の間に空行を挟むと、それぞれがパラグラフ(p)となります。
   <br/>
  空行をはさまずに次の文を書くと、パラグラフ(p)内で改行(br)されます。</p>
  <p>チルダを使うと強制的に改行されます。
   <br/>
  これはグラフ要素などで使用することを意図しています。 <br/>
   <br/>
   <br/>
   <br/>
  通常の文では使う必要はありません。</p>
  """

  def test_wiki2html():
      r = Wiki2Html().parse('')
      eq_(r, '')

      r = Wiki2Html().parse(wiki)
      eq_(r, result)

      r = Wiki2Html().parse(wiki.encode('utf-8'))
      eq_(r, result.encode('utf-8'))
{% endraw %}
{% endhighlight %}


## 実装

### すべてencoded-stringでがんばる実装

- web, file, commandlineからの入力をは、たいていもともとencoded-stringである。
- 汚染されていない関数、encoded-stringに汚染された関数を使って、各種文字列処理を行う。
- web, fileなどに出力するときは、そのまま書けばOK (utf-8になる)

### すべてunicodeでがんばる実装

- web, file, commandlineからの入力(encoded-string)をすべてunicodeに変換する。
- 汚染されていない関数、unicodeに汚染された関数を使って各種文字列処理を行う
- web, fileなどに出力するときは、encoded-string(utf-8)などに変換して書く。

内部でunicode文字列を使わないなら、すべてencoded-stringでがんばる実装よりも非効率。

### すべてencoded-stringでがんばる実装からすべてunicodeでがんばる実装に書き換える

- web, file, commandlineへの入力境界にて、unicodeに変換する
- 汚染されていない文字列処理関数はそのまま使える。
- 汚染された関数は、なるべく汚染されていない関数に書き直す。
- 残りのencoded-stringに汚染された関数をunicodeに汚染された関数に書き換える
  日本語文字列が含まれるリテラルを探しては頭にuをつける作業(例: 'ほげ'->u'ほげ')
- 出力境界にて、encoded-string(utf-8)に変換する

## Python 3

Python 3では文字列周りの仕様が抜本的に変更され、新stringはすべてunicodeになり、従来のencoded-stringはbytes(可変長はbytearray)というクラスに分けられた。

- Python 3 string = Python 2 unicode + ascii-string
- Python 3 bytes/bytearray = Python 2 encoded-string

すべてunicodeでがんばる実装にしておいたほうが、Python3への移植の時は手間が省ける。
