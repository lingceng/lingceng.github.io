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

It's fast because no need to recompile the JavaScript or CSS.

As you can see, we must keep stuffs in `<head>` the same between pages
when jumping with Turbolink.  In other words, Things will go wrong if the pages have
different content in `<head>`.

## What's the best practice?

Put all JavaScript and CSS in `<HEAD>` and keep them the same in every page.
Use [jquery.turbolink](https://coderwall.com/p/ypzfdw/faster-page-loads-with-turbolinks) to
hijack `jQuery.ready()` in jQuery like following:

    jQuery
    jQuery.turbolinks
    ...other scripts go here...
    Turbolinks

[Here](https://coderwall.com/p/ypzfdw/faster-page-loads-with-turbolinks)
explains more:

> The reason for jQuery.turbolinks being before all scripts is so to let
> it hijack the `$(function() { ... })` call that your other scripts will use.
>
> Turbolinks then needs to be at the end because it has to be the last
> to install the click handler, so not to interfere with other scripts.

## Take care of some dangerous things
Global delegated events will effect every page. eg. You add a script as
following and "clicked button" will be printed when you click a button
in every pages.

    $(document).on('click', 'button', function(){
      console.log("clicked button")
    })

Global `setInterval` or `setTimeout` need to be clear too! DEMO:

    $(document).one('page:before-change', function(event) {
      clearTheTimer();
    }

See more [here](http://staal.io/blog/2013/01/18/dangers-of-turbolinks/)

## How to control javascript init for every page?

Mark to `<body>` when need specific page logic, trigger it according to mark.
I want javascript only run in topics pages:

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

## You can put javascript at the end of body. But it's not recommended.
You will loose benefit with HTTP cache when you put javascript at the and of the
body.

## Turbolink has other features to make it fast.

Turbolink uses client cache pages.

    # View current page cache size
    Turbolinks.pagesCached();
    # Set the cache size
    Turbolinks.pagesCached(20);

Turbolink uses transition cache.
Transition will immediately display the cached copy of page and then replace
with remote returned one.

    # enable transition cache
    Turbolinks.enableTransitionCache();

Browser native process bar will not work when using Turbolink.
So Turbolink provide a JavaScript-and-CSS-based one. eg. in 2.5 version

    Turbolinks.enableProgressBar();

## What will happen when I changed head content?

It depends on how you change it.

If you add some `<script type="text/javascript">` tag in head, eg.

    // On page B
    <script type="text/javascript">
      console.log('hello')
    </script>

When you click a link on page A and jump to page B, the 'hello' will **not** printed.
The script tag will be ignored.

It's the same when you make it a `src` link.

    // On page B
    <script type="text/javascript" src='hello.js'> </script>

You add a `data-turbolinks-track` tag to make it work. But it has drawbacks.

    <script type="text/javascript" src="/hello.js" data-turbolinks-track></script>

When this case, 'hello' will be printed,
every things seems fine except **slow page load**.

You'll technically be requesting the same page twice.
Once through Turbolinks to detect that the assets changed,
and then again do a full redirect to that page.

You should always add `data-turbolinks-track` to JavaScript and CSS links.
This will trigger full page load when your assets changed.

When page A and page B have different `track targets`,
every switch between them will cause `double load`.

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
