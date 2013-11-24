---
layout: post
title: "Blog設置"
description: ""
category: ""
tags: [jekyll]
---
{% include JB/setup %}


- [Jekyll boostrap](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)
- pygmentsをインストール。syntax.cssを設置。
- jekyll serve --watch でチェック
- Social box <http://128bit.blog41.fc2.com/blog-entry-333.html>

"_includes/themes/twitter/default.html"

{% highlight html %}

<div id="fb-root"></div>
<script>(function(d, s, id) {
  var js, fjs = d.getElementsByTagName(s)[0];
  if (d.getElementById(id)) return;
  js = d.createElement(s); js.id = id;
  js.src = "//connect.facebook.net/ja_JP/all.js#xfbml=1";
  fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));</script>

...
{% endhighlight %}

post.html

{% highlight html %}
{% raw %}
{% capture fullpath %}{{site.production_url}}{{page.url}}{% endcapture %}

<div class="social-button">
    <ul class="social-button">
        <li><a href="http://b.hatena.ne.jp/entry/{{fullpath}}" class="hatena-bookmark-button" data-hatena-bookmark-title="{{page.title}}" data-hatena-bookmark-layout="standard" title="このエントリーをはてなブックマークに追加"><img src="http://b.st-hatena.com/images/entry-button/button-only.gif" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="http://cdn-ak.b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script></li>
        <li><div class="fb-like" data-href="{{fullpath}}" data-send="false" data-layout="button_count" data-width="130" data-show-faces="false"></div></li>
        <li><a href="https://twitter.com/share" class="twitter-share-button" data-url="{{fullpath}}" data-text="{{page.title}}" data-lang="jp" data-count="horizontal">Tweet</a><script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script></li>
        <li><script type="text/javascript" src="https://apis.google.com/js/plusone.js"></script><div class="g-plusone" data-size="medium" data-href="{{fullpath}}"></div></li>
    </ul>
</div>
{% endraw %}
{% endhighlight %}

