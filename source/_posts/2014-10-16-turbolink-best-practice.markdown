---
layout: post
title: "Turbolink Best Practice"
date: 2014-10-16 12:12
comments: true
categories: [Rails, Ruby]
---

## What does Turbolink do?

[Turbolink](https://github.com/rails/turbolinks) makes browser to only replace
page's `<body>` and `<title>` to simulate page jumping when click a link.

It's fast because no need to recompile the JavaScript and CSS.
As you can see, we must keep stuffs in head the same, or things will go wrong.

#### Turbolink has other features make it fast

Use client cache pages.

    # View current page cache size
    Turbolinks.pagesCached();
    # Set the cache size
    Turbolinks.pagesCached(20);

Use transition cache.
Transition will immediately display the cached copy of page and then replace
with remote returned one.

    # enable transition cache
    Turbolinks.enableTransitionCache();


Browser native process bar will not work when using Turbolink.
So Turbolink provide a JavaScript-and-CSS-based one.

    Turbolinks.enableProgressBar();

## What's the best practice?

+ ** Never change `<HEAD>` content except `title` **
+ ** Put all JavaScript and CSS in `<HEAD>` **
+ ** Use `jquery.turbolink` to fix `DOMContentLoaded` or `jQuery.ready()` in jQuery. **

## You want some JavaScript only included in some page?

Mark to `<body>` when need specific page logic, trigger it according to mark, eg.

    // In app/views/layouts/application.html.erb
    <body data-controller-name="<%= controller_name %>">

    // In topics.coffee
    window.Topics =
      replies_per_page: 50
      init : () ->
        console.log "hello"

    $(document).ready ->
      if $('body').data('controller-name') in ['topics']
        Topics.init()

    // Add topics to appliction.js
    //= require topics


Note: This will also avoid `config.precompile.assets+=%w{user}` in Rails.

## What will happen when add JavaScript in page body?

You should not add JavaScript to `<body>`, they will be re-evaluated.
Especially when doing event bind, it will cause event binding multiple times.

Note that you can add `data-turbolinks-eval=false` to prevent re-evaluating.

    <script type="text/javascript" data-turbolinks-eval=false>
      console.log("I'm only run once on the initial page load");
    </script>

## What will happen when I changed head content?

It depends on how you change it.

If you add some `<script type="text/javascript">` tag, eg.

    // On page B
    <script type="text/javascript">
      console.log('hello')
    </script>

When you click a link on page A to page B, the 'hello' will **not** printed.
The script tag will be ignored.

It's the same when you make it a `src` link.

    // On page B
    <script type="text/javascript" src='hello.js'> </script>

Only if you add a `data-turbolinks-track` tag.

    <script type="text/javascript" src="/hello.js" data-turbolinks-track></script>


When this case, 'hello' will be printed,
every things seems fine except **slow page load**.

You'll technically be requesting the same page twice.
Once through Turbolinks to detect that the assets changed,
and then again do a full redirect to that page.

You should always add `data-turbolinks-track` to JavaScript and CSS links.
This will trigger full page load when your assets changed.
It is only useful to clear cache in a page instance.

When page A and page B have different `track targets`,
every switch between them will cause `double load`.
So, you'd better not rely on `data-turbolinks-track` to jumps between different head content.

See the [code](https://github.com/rails/turbolinks/blob/master/lib%2Fassets%2Fjavascripts%2Fturbolinks.js.coffee#L231)
to know the details

{% codeblock lang:coffeescript %}
extractTrackAssets = (doc) ->
  for node in doc.querySelector('head').childNodes when node.getAttribute?('data-turbolinks-track')?
    node.getAttribute('src') or node.getAttribute('href')

assetsChanged = (doc) ->
  loadedAssets ||= extractTrackAssets document
  fetchedAssets  = extractTrackAssets doc
  fetchedAssets.length isnt loadedAssets.length or intersection(fetchedAssets, loadedAssets).length isnt loadedAssets.length
{% endcodeblock %}

One last rescue is to prevent turbolink jump by add `data-no-turbolink` tag.
And then you will not benefit from turbolink speed boost.

{% codeblock lang:html %}
<a href="/">Home (via Turbolinks)</a>
<div id="some-div" data-no-turbolink>
  <a href="/">Home (without Turbolinks)</a>
</div>
{% endcodeblock %}

