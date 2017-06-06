---
layout: post
title: "Put javascript library files together in Rails"
date: 2015-12-15 17:06
comments: true
categories: [rails, javascript]
---

**TLDR: Put all javascript library files under `app/assets/libfolder`.
Refer the files without the `libfolder`**

We always need to import some javascript libs to our Rails application.
Some libs has correspondent gems.
For example, **jquery.js** has a **jquery-rails gem**.

But some other libs may not have the correspondent gems.
What's more, it may be a waste to use a "wrapper gem" to just import a javascript file.
Some wrapper gem may not up-to-date to the origin javascript lib.
So we need to import javascript libs manually in some cases.

Rails default use [sprockets](https://github.com/sstephenson/sprockets) to
manage assets.
And we always split assets into `javascripts`, `stylesheets`, `images` and `fonts` folders.
But if we split a javascript library into these files, it maybe a little messy.

**Actually we can put the files together in a same foler.**

For example, I want to import [bootstrap-sweetalert](https://github.com/lipis/bootstrap-sweetalert)  into
my Rails app.
I download the `sweet-alert.less` and `sweet-alert.js`.
Then put the two files into `vender/assets/sweet-alert`.
`sweet-alert` here is a new created folder.

The `vender/assets/sweet-alert` folder will be treat as base path to query
assets. So we can refer the `sweet-alert.js` in our application.js as following:

     //= require sweet-alert

We can do the same for `sweet-alert.less` in our application.less.

Note that we have [other ways to manage the assets](https://www.codefellows.org/blog/5-ways-to-manage-front-end-assets-in-rails). But it's another topic then.
