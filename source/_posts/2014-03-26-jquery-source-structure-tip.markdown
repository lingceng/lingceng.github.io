---
layout: post
title: "jQuery Source Structure Tip"
date: 2014-03-26 09:12
comments: true
categories:
---

I'm puzzled when I first see the jQuery source code.
I finaly find a clue after several attempts.

Put aside other modules, we can find out that jQuery has the ordinary JS class structrue.
{% codeblock lang:javascript %}
jQuery = function() {

}
jQuery.prototype = {

}
{% endcodeblock %}

It seems not that same with how we use jQuery. secret is the aliases.
{% codeblock lang:javascript %}
// we can use the dollar sign to call the jQuery function
$ = jQuery;

// we can use the `jQuery.fn.newFeature = function() {}` to extend the jQuery instance function
jQuery.fn = jQuery.prototype;
{% endcodeblock %}

But when `$("#someid")` always return a jQuery instance, where the `new` keyword?
{% codeblock lang:javascript %}
jQuery = function() {
    // jQuery.prototype.init is the actually creator function
    return new jQuery.fn.init( selector, context );
}

//  mount prototype on jQuery to jQuery.fn.init
jQuery.fn.init.prototype = jquery.fn
{% endcodeblock %}


Let's see the complete struture now.
{% codeblock lang:javascript %}
// utils

// class structure
jQuery = function() {
    return new jQuery.fn.init( selector, context );
}
jQuery.fn = {
   init: function(selector, context) {}
   // other core instance methods
}
jQuery.fn = jQuery.prototype;
jQuery.fn.init.prototype = jquery.fn

// instance methods extend
// or use this style: jQuery.fn.hello = funtion() {}
jQuery.extend(jQuery.fn, {
   // ...
});

// class methods extend
// or use this style: jQuery.hello = funtion() {}
// maybe some utils
jQuery.extend(jQuery, {
  isFunction: function( obj ) {
    return jQuery.type(obj) === "function";
  }
  // ...
});

// module export or AMD
window.$ = window.jQuery = jQuery
{% endcodeblock %}


