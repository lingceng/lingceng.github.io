---
layout: post
title: "Turbolink Best Practice"
date: 2014-10-16 12:12
comments: true
categories:
---

## What does [turbolinks](https://github.com/rails/turbolinks) do?

Only replace the page `<body>` and `<title>` with Get respond when click a link.

It's fast because no need to recompile the JavaScript and CSS between
each page change.

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


Browser native process bar will not work when using turbolink.
So turbolink provide a JavaScript-and-CSS-based one.

    Turbolinks.enableProgressBar();

## What's the best practice?

+ Use `jquery.turbolink` to fix Dom ready event in jQuery.
+ Put all JavaScript and CSS in `<HEAD>`

#### Has javascripts only needed in some page?

Mark tag to `<body>` when need specific page logic.
Trigger it according to the tag value.

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

## What will happen when add JavaScript in body?

You should not add JavaScript to `<body>`.
Especially when bind event, it will cause event bind multiple times.

Note that you can add `data-turbolinks-eval=false` to rescue this.

    <script type="text/javascript" data-turbolinks-eval=false>
      console.log("I'm only run once on the initial page load");
    </script>

## What will happen when I changed head content?

It depends on how you change it.

If you add some `<script type="text/javascript">` tag, like something like this:

    // On page B
    <script type="text/javascript">
      console.log('hello')
    </script>

When I click a link on page A to page B, the 'hello' will **not** printed.
The script tag will be ignored.
It's the same when you add a `src` to script tag.

Things will change when this case

    <script type="text/javascript" src="/hello.js" data-turbolinks-track></script>

`data-turbolinks-track` make turbolink track the change in head.

When this case, 'hello' will be printed,
every things seems fine except slow page load.
When this happens, you'll technically be requesting the same page twice.
Once through Turbolinks to detect that the assets changed,
and then again when we do a full redirect to that page.

You should always add `data-turbolinks-track` to JavaScript and CSS links.
This will trigger full page load when your assets changed.
But when two pages have different `track targets`, every switch between the two
pages will cause `double load`.

See the
[code](https://github.com/rails/turbolinks/blob/master/lib%2Fassets%2Fjavascripts%2Fturbolinks.js.coffee#L231)
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

In this situation, you can add tag to prevent turbolink jump.

{% codeblock lang:html %}
<a href="/">Home (via Turbolinks)</a>
<div id="some-div" data-no-turbolink>
  <a href="/">Home (without Turbolinks)</a>
</div>
{% endcodeblock %}

But you must add to "every in and out link" for page B, that's not a easy job.

## Conclusion
### Never change head contents and follow the best practice.
