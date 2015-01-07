---
layout: post
title: "Use Timezone In Rails"
date: 2014-11-22 18:38
comments: true
categories: [Rails, Ruby]
---

#### Create Table and config
Let's create a table first.

    class CreateHellos < ActiveRecord::Migration
      def change
        create_table :hellos do |t|
          t.timestamps
        end
      end
    end

Add following to `config/application.rb` to config timezone.

    Rails.application.config.active_record.default_timezone = :local
    Rails.application.config.time_zone = 'Beijing'

    Rails.application.config.i18n.default_locale = 'zh-CN'

#### `to_s(:db)` vs `to_s`

    > l = FinanceItem.create
    #=> #<FinanceItem id: 1, created_at: "2014-11-22 03:00:32", updated_at: "2014-11-22 03:00:32" >
    > l.created_at
    #=> Sat, 22 Nov 2014 11:00:32 CST +08:00
    > l.created_at.to_s(:db)
    #=> "2014-11-22 03:00:32"

`to_s(:db)` will output in time UTC which is always not us want.

    TimeWithZone to_s(:db)
    :db format outputs time in UTC;
    all others output time in local

#### `default_timezone` is so necessary

`config.time_zone` sets the default time zone for the application
and enables time zone awareness for Active Record.

`config.active_record.default_timezone` determines whether to use Time.local
 (if set to :local) or Time.utc (if set to :utc) when pulling dates and times
from the database.  The default is :utc.

    `utc()`
    Returns a Time or DateTime instance that represents the time in UTC.

    `local()`
    Method for creating new ActiveSupport::TimeWithZone instance
    in time zone of self from given values.

    Time.zone = 'Hawaii'                    # => "Hawaii"
    Time.zone.local(2007, 2, 1, 15, 30, 45) # => Thu, 01 Feb 2007 15:30:45 HST -10:00

#### DateTime.parse vs Time.zone.parse

    > a = DateTime.parse('2014-11-22 12:35:05')
    #=> Sat, 22 Nov 2014 12:35:05 +0000
    > a.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 12:35:05 +0000"
    > a.in_time_zone.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 20:35:05 +0800"

    > b = Time.zone.parse('2014-11-22 12:35:05')
    #=> 2014-11-22 12:35:05
    > b.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 12:35:05 +0800"
    > c = '2014-11-22 12:35:05'.to_time
    #=> 2014-11-22 12:35:05
    > c.to_s
    #=> "2014-11-22 12:35:05 +0800"

Notice that `Time.zone.parse` has `+0800` timezone offset.

### Time.now vs Time.current

Let's see the definition of `Time.current`

    # File activesupport/lib/active_support/core_ext/time/calculations.rb, line 29
    def current
      ::Time.zone ? ::Time.zone.now : ::Time.now
    end

Time.now uses the system time because it's is part of the Ruby standard library.
Time.zone.now will set time with `config.time_zone`.

### In a word
Set the following config:

    active_record.default_timezone = :local
    # or others
    time_zone = 'Beijing'

Use `Time.zone.parse` do **not** use `DateTime.parse`.
Use `Time.zone.now` or `Time.current` do **not** use `Time.now`.
