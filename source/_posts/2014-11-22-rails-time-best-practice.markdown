---
layout: post
title: "Rails Time Best Practice"
date: 2014-11-22 18:38
comments: true
categories: [Rails, Ruby]
---

#### Let's create demo table and config first

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

`config.time_zone` sets the default time zone for the application
and enables time zone awareness for Active Record.

`config.active_record.default_timezone` determines whether to use Time.local
 (if set to :local) or Time.utc (if set to :utc) when pulling dates and times
from the database.  The default is :utc.

#### Rails `to_s(:db)` vs `to_s`

Run codes in rails console.

    l = FinanceItem.create
    #=> #<FinanceItem id: 1, created_at: "2014-11-22 03:00:32", updated_at: "2014-11-22 03:00:32" >
    l.created_at
    => Sat, 22 Nov 2014 11:00:32 CST +08:00

The timezone offset is **+08:00** because I set `config.time_zone = 'Beijing'`

    l.created_at.class
    #=> ActiveSupport::TimeWithZone

Rails use ActiveSupport::TimeWithZone for datetime field.


    > l.created_at.strftime("%Y-%m-%d %H:%M%S")
    #=> "2014-11-22 11:00:32"
    > l.created_at.to_s(:db)
    #=> "2014-11-22 03:00:32"

We can see  two date string diff. Because `to_s(:db)` output time string in UTC.
`strftime` or `to_s` ouptut time string with configured time zone.

Here is the `TimewithZone#to_s(format)` source:

    :default - default value, mimics Ruby 1.9 Time#to_s format.
    :db - format outputs time in UTC :db time. See Time#to_formatted_s(:db).
    Any key in Time::DATE_FORMATS can be used. See active_support/core_ext/time/conversions.rb.
    # File activesupport/lib/active_support/time_with_zone.rb, line 193
    def to_s(format = :default)
      if format == :db
        utc.to_s(format)
      elsif formatter = ::Time::DATE_FORMATS[format]
        formatter.respond_to?(:call) ? formatter.call(self).to_s : strftime(formatter)
      else
        "#{time.strftime("%Y-%m-%d %H:%M:%S")} #{formatted_offset(false, 'UTC')}" # mimicking Ruby 1.9 Time#to_s format
      end
    end

However when subject is **Time**, `to_s(:db)`  and `strftime("%Y-%m-%d %H:%M:%S")`
outputs the same.

    a = Time.now
    => 2015-03-12 16:56:21
    a.zone
    => "CST"
    a.to_s(:db)
    => "2015-03-12 16:56:21"
    a.strftime("%Y-%m-%d %H:%M:%S")
    => "2015-03-12 16:56:21"

As we can see, `to_s(:db)` is NOT consistent between Time and TimeWithZone.
Calling `to_s(:db)` on TimeWithZone is always error-prone unless you know what
you are doing.
So you'd better NOT use `to_s(:db)`.

#### DateTime.parse vs Time.zone.parse

    > a = DateTime.parse('2014-11-22 12:35:05')
    #=> Sat, 22 Nov 2014 12:35:05 +0000
    > a.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 12:35:05 +0000"
    > a.in_time_zone.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 20:35:05 +0800"

    b = Time.zone.parse('2014-11-22 12:35:05')
    #=> 2014-11-22 12:35:05
    b.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 12:35:05 +0800"

Notice that `Time.zone.parse` has `+0800` timezone offset.

### Some time tip

    time = '2014-11-22 12:35:05'.to_time
    #=> 2014-11-22 12:35:05
    time.to_s
    #=> "2014-11-22 12:35:05 +0800"
    time.class
    #=> Time

Use `to_time` in rails will consider timezone offset.

    time.zone
    #=> "CST"

Here CST stand for (China Standard Time)

Checkout time zone name and offset

    time.strftime('%Z')
    #=> "CST"
    time.strftime('%z')
    #=> "+0800"

### Time.now vs Time.current

Let's see the definition of `Time.current`

    # File activesupport/lib/active_support/core_ext/time/calculations.rb, line 29
    def current
      ::Time.zone ? ::Time.zone.now : ::Time.now
    end

Time.now uses the **system time zone** because it's is part of the Ruby standard library.
Time.zone.now will set zone with `config.time_zone`.

Using `Time.now` make troubles when your system time zone is different with
`config.time_zone`

### Best practice
Set the following config.

    config.active_record.default_timezone = :local
    config.time_zone = 'YourLocalName'

Use `Time.zone.parse` and do **NOT** use `DateTime.parse`.
Use `Time.zone.now` or `Time.current` and do **NOT** use `Time.now`.
Thus we can keep all time class to TimeWithZone and get consistent behavior.
