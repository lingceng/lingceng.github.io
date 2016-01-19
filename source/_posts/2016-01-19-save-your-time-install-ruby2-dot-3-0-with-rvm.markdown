---
layout: post
title: "Save Your Time: Install ruby2.3.0 with RVM"
date: 2016-01-19 12:24
comments: true
categories: ['Ruby']
---

Note that:  **ruby-2.3.0** is not present now when run `rvm list known` command in rvm 1.26.11.  
It may be available in next rvm version.
See [here](https://github.com/rvm/rvm/blob/1.26.11/config/known#L11) for more details.

But you can just run the following directly.

    rvm install ruby-2.3.0

And set 2.3.0 as default ruby version

    rvm --default use 2.3.0

Copy gems from older version

    rvm gemset copy 2.1.5 2.3.0

Get the ruby version and celebrate!

    ruby -v
    # ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-darwin15]

Try new features in irb

    puts 'hi'&.upcase

    h = { a: 1, b: 2, c: 3, d: 4 }
    [ :a, :c, :d, :b ].map(&h)


