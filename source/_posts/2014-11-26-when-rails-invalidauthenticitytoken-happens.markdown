---
layout: post
title: "When Rails InvalidAuthenticityToken Happens"
date: 2014-11-26 16:31
comments: true
categories: [rails]
---

`InvalidAuthenticityToken` happens if you enabled `protect_from_forgery`
but does not handle token well.

If you use the jquery-ujs library the content of that meta tag is automatically
added (as a request header) to any ajax requests made.

`jquery-ujs` add a ajax filter to append token to parameter

    // ajaxPrefilter is a jquery method
    $.ajaxPrefilter(function(options, originalOptions, xhr) {
      if ( !options.crossDomain ) { rails.CSRFProtection(xhr); }
    });

Note: Only HTML and JavaScript requests are checked.
See more about `ActionController::RequestForgeryProtection` in rails API.

### Conclusion

Always import `jquery-ujs` when enabled `protect_from_forgery`
And put `<%= csrf_meta_tags %>` in page head.
