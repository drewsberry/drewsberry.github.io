---
layout: post
title: Jekyll KaTeX Block
permalink: jekyll-katex-block
comments: True
categories: coding
live: true
---

If you follow [Hacker News](https://news.ycombinator.com/item?id=8320439), then you might've seen the latest development from Khan Academy: [{% katex %}](http://khan.github.io/KaTeX/).

Although it hasn't yet implemented the full set of mathematical symbols, {% katex %} promises to replace MathJax as *the* web technology for displaying mathematical expressions. This plugin utilises Jekyll's plugin system to enable you to easily add mathematical equations to your statically generated site, by adding a Liquid block corresponding to content to be rendered by {% katex %}.

It's as simple as adding this plugin to your `_plugins` folder, pointing the plugin to your {% katex %} JavaScript file in `_config.yml`, and including the {% katex %} CSS in the resulting web pages.

For more information, continue reading, or if you want to just go straight to the code, head over to [https://github.com/drewsberry/jekyll-katex-block](https://github.com/drewsberry/jekyll-katex-block).

<!--more-->

Why KaTeX?
----------

So why should you use {% katex %}? Well, here's a few reasons:

* It's super fast;
* It supports server side rendering;
* Pure CSS (the JS is only necessary is you're rendering client-side);
* It's [*super*](http://jsperf.com/katex-vs-mathjax) fast.

The Jekyll KaTeX plugin
-----------------------

So how does this Jekyll plugin fit in?

Well,{% comment %}as I explain in a [future post](http://drewsilcock.co.uk/jekyll-css-compressor),{% endcomment %} there are three types of Jekyll plugins:

1. **Generators**
2. **Converters**
3. **Tags**

A subset of the **Tags** plugin is the **Block** plugin. What this does is provides access to a [Liquid block](https://github.com/Shopify/liquid/wiki/Liquid-for-Programmers#create-your-own-tag-blocks), that takes the content inside the block and does with it as you specify in your plugin. Here's an example of a Liquid block:

{% highlight liquid lineanchors %}{% raw %}
{% myblock %}
This is the content inside the block!
{% endmyblock %}
{% endraw %}{% endhighlight %}

The Jekyll {% katex %} block then provides the `{% raw %}{% latex %}{% endraw %}` block, which uses {% katex %} to compile the contents of the block into an equation. So this is how you'd use it in your post:

{% highlight liquid lineanchors %}{% raw %}
{% latex %}
E = \gamma mc^2
{% endlatex %}
{% endraw %}{% endhighlight %}

In addition, you can pass `tokens` to blocks like so:

{% highlight liquid lineanchors %}{% raw %}
{% myblock mytoken %}
Block contents
{% endmyblock %}
{% endraw %}{% endhighlight %}

So if you pass the `centred` token to the `latex` block like so:

{% highlight liquid lineanchors %}{% raw %}
{% latex centred %}
E = \gamma mc^2
{% endlatex %}
{% endraw %}{% endhighlight %}

Then the equation will also be cented on the page, with some space above and below. This is achieved using a little bit of inline CSS injected into the equation.

How to install it
-----------------

If you've already got the {% katex %} JavaScript installed on your system, as well as the CSS files and the font files, then all you need to do is drop [jekyll-katex-block.rb](https://raw.githubusercontent.com/drewsberry/jekyll-katex-block/master/katex_block.rb) into your `_plugins` directory in your project, and you're ready to go!

Just make sure that you're loading the CSS in your web page (but *not* the JavaScript) and have the font files available.

Options
-------

You can specify where to tell the plugin to look for your JavaScript file in your `_config.yml` with the following:

{% highlight yaml lineanchors %}
katex:
    path_to_js: "./the/path/to/your/js"
{% endhighlight %}

Troubleshooting
---------------

If you're seeing a bunch of weird symbols instead of your equations, then it's probably because you're not using UTF-8. This is what it'll look like:

<img src="/public/media/jekyll-katex-block/katex_no_utf8.png" alt="without utf-8, your browser will get very confused" class="ds-centred">

Put the following in the head of your HTML to sort it out:

{% highlight html lineanchors %}
<meta charset=utf-8>
{% endhighlight %}

If you're seeing your equations, but they don't have any special formatting and just look like weird, squished, ugly versions of what you want, like this:

<img src="/public/media/jekyll-katex-block/no_font_equation.png" alt="this is what happens if you don't have the font files in the right place" class="ds-centred">

Then it's probably because your server can't find the font files. You can test this out by doing the following:

{% highlight console lineanchors %}
$ jekyll build
$ cd _site {% comment %} _h {% endcomment %}
$ python -m SimpleHTTPServer
{% endhighlight %}

Go to your web page in question with your web browser, and look at the output of your Python session. It should say something like:

{% highlight pycon lineanchors %}
Serving HTTP on 0.0.0.0 port 8000 ...
127.0.0.1 - - [04/Nov/2014 12:18:07] "GET / HTTP/1.1" 200 -
{% endhighlight %}

But if your server can't find the font files, it'll output the following:
{% highlight pycon lineanchors %}
127.0.0.1 - - [04/Nov/2014 12:23:26] code 404, message File not found
127.0.0.1 - - [04/Nov/2014 12:23:26] "GET /fonts/KaTeX_Math-Italic.woff HTTP/1.1" 404 -
127.0.0.1 - - [04/Nov/2014 12:23:26] code 404, message File not found
127.0.0.1 - - [04/Nov/2014 12:23:26] "GET /fonts/KaTeX_Size1-Regular.woff HTTP/1.1" 404 -
127.0.0.1 - - [04/Nov/2014 12:23:26] code 404, message File not found
127.0.0.1 - - [04/Nov/2014 12:23:26] "GET /fonts/KaTeX_Size3-Regular.woff HTTP/1
{% endhighlight %}

To solve this, put all your {% katex %} font files in in the same directory as your {% katex %} CSS file, that way it'll automatically look for and find it. If you're still having problems, have a look at the example [Jekyll site](https://github.com/drewsberry/jekyll-katex-block/tree/master/test) on the GitHub repo, or comment below.

Now you should be back to how it's supposed to look:

{% latex centred %}
K_0 = \sqrt{\frac{m}{ih t}} e^{-\frac{m(x-x')^2}{i\frac{h}{2\pi} t}}
{% endlatex %}

How it works
------------

The plugin basically works by compiling {% katex %} within the Ruby script using the [ExecJS](https://rubygems.org/gems/execjs) module. The JavaScript function `renderToString` is then called from this compiled JavaScript on the content inside the block. This converts the content from {% latex %} \LaTeX {% endlatex %} to HTML.

The Ruby code to actually do this looks like this:

{% highlight ruby lineanchors %}
# Compile the KaTeX JavaScript
katexsrc = open(path_to_katex_js).read
@katex = ExecJS.compile(katexsrc)

# Turn a LaTeX string into HTML equation
equation_string = "E = \gamma mc^2"
html = @katex.call("katex.renderToString", equation_string)
{% endhighlight %}

The `@` sign indicates that the `katex` variable is an instance variable, which basically means that it's available to all the methods within the current class. The block itself is a class inheriting from `Liquid::Block`. It's a member of the `Jekyll::Tags` module, which makes it accessible to Jekyll on runtime.

As with all tags, it then needs to be registered. This is where you tell Liquid what identifier the tag should have, i.e. what string inside the {% raw %}{% and %}{% endraw %} brackets should activate the plugin. This looks like the following:

{% highlight ruby lineanchors %}
Liquid::Template.register_tag('latex', Jekyll::Tags::KatexBlock)
{% endhighlight %}

Where `KatexBlock` is the {% katex %} block class.

To see the full code, head over to the [GitHub repository](https://github.com/drewsberry/jekyll-katex-block).
