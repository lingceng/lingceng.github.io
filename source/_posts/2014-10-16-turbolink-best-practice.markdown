---
layout: post
title: "Turbolink Best Practice"
date: 2014-10-16 12:12
comments: true
categories: [rails, ruby]
---

TLDR: Take care of global variables, global events binding.

### What does Turbolink do?

[Turbolink](https://github.com/rails/turbolinks) 
makes browser only replace page's `<body>` and `<title>` to simulate page jumping.  
So Javascript context will not change even when page jumps.

### What's the best practice?
Put all JavaScript and CSS in `<HEAD>` and keep them the same for every page.  
Mark page to separate logic:

    // In app/views/layouts/application.html.erb
    <body data-controller-name="<%= controller_name %>">

    $(document).ready ->
      if $('body').data('controller-name') in ['topics']
        console.log "Only run in topics page"

### Another practice
If you are maintaining a ERP like system.  
The javascript may diffs from page to page, you can put javascript at the end of body.

    <body>
      <script type="text/javascript" src="/topics.js"></script>
    <body>

### Fix jQuery ready
Use [jquery.turbolink](https://coderwall.com/p/ypzfdw/faster-page-loads-with-turbolinks)
to hijack `jQuery.ready()`.  
It guarantee all events binded with `jQuery.ready()` are triggered, 
no matter how you do a page jump, a html page load or  a turbolink jump.

You should load scripts in following order:

    jQuery
    jQuery.turbolinks
    ...other scripts go here...
    Turbolinks

[Here](https://coderwall.com/p/ypzfdw/faster-page-loads-with-turbolinks) explains: 

> The reason for jQuery.turbolinks being before all scripts is so to let
> it hijack the `$(function() { ... })` call that your other scripts will use.
>
> Turbolinks then needs to be at the end because it has to be the last
> to install the click handler, so not to interfere with other scripts.

### Take care of dangerous global handles
Global delegated events will effect every page.
eg. You add a script as following:

    $(document).on('click', 'button', function(){
      console.log("clicked button")
    })

"clicked button" will be printed when you click a button **in every pages**.

Another example is the hijacked jquery ready event as mentioned above.
So you should distinguish pages in you codes. eg.

    $(function() {
      if (current_page == 'index') {
        // do staff for index page
      }
    });

See details about distinguishing pages below.

Global `setInterval` or `setTimeout` need to be clear too! 

    $(document).one('page:before-change', function(event) {
      clearTheTimer();
    }

See more [here](http://staal.io/blog/2013/01/18/dangers-of-turbolinks/)

### What will happen when I changed head content?

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

You can add a `data-turbolinks-track` tag to make it work. But it has drawbacks.

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
