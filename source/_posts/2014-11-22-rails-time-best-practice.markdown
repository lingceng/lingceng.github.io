---
layout: post
title: "Rails Time Best Practice"
date: 2014-11-22 18:38
comments: true
categories: [Rails, Ruby]
---

#### Setup

Let's create a demo table and do some configurations in a rails project.
Then we'll do some tests about time in `rails console`.

    class CreateHellos < ActiveRecord::Migration
      def change
        create_table :hellos do |t|
          t.timestamps
        end
      end
    end

Add following to `config/application.rb`

    Rails.application.config.active_record.default_timezone = :local
    Rails.application.config.time_zone = 'Beijing'

`config.time_zone` sets the default time zone for the application
and enables time zone awareness for Active Record.

`config.active_record.default_timezone` determines whether to use Time.local
 (if set to :local) or Time.utc (if set to :utc) when pulling dates and times
from the database.  The default is :utc.

#### `to_s(:db)` is error-prone

Run following codes in rails console.
Here CST stands for (China Standard Time) which is zone name for Beijing.

    l = FinanceItem.create
    #=> #<FinanceItem id: 1, created_at: "2014-11-22 03:00:32", updated_at: "2014-11-22 03:00:32" >
    l.created_at
    #=> Sat, 22 Nov 2014 11:00:32 CST +08:00

The timezone offset is **+08:00** because I set `config.time_zone = 'Beijing'`

    l.created_at.class
    #=> ActiveSupport::TimeWithZone

Rails use [ActiveSupport::TimeWithZone](http://api.rubyonrails.org/classes/ActiveSupport/TimeWithZone.html#method-i-to_s) for datetime field.

    l.created_at.strftime("%Y-%m-%d %H:%M%S")
    #=> "2014-11-22 11:00:32"
    l.created_at.to_s(:db)
    #=> "2014-11-22 03:00:32"

We can see, two date strings are different.
Because `to_s(:db)` always output time string in UTC.
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

How about `Time#to_s(:db)` and `Time#strftime("%Y-%m-%d %H:%M:%S")`?
Here zone name is CST because my system local is 'Beijing'.
It's different with TimeWithZone which get zone from project configurations `config.time_zone`.

    a = Time.now
    => 2015-03-12 16:56:21
    a.zone
    => "CST"
    a.to_s(:db)
    => "2015-03-12 16:56:21"
    a.strftime("%Y-%m-%d %H:%M:%S")
    => "2015-03-12 16:56:21"

`Time#to_s(:db)` and `Time#strftime("%Y-%m-%d %H:%M:%S")` output the same.
We can find out that `Time#to_s(:db)` is actually called Time#strftime('%Y-%m-%d %H:%M:%S') from source

As we can see, `to_s(:db)` is NOT consistent between Time and TimeWithZone.
TimeWithZone#to_s(:db) will generate time string of UTC.
Time#to_s(:db) will return time string of configured local.

So calling `to_s(:db)` is always error-prone. Do not use it unless you know what you are doing.
And you'd better not use Time and TimeWithZone interchangeably.

#### Time.zone.parse get time with timezone
Let's try following codes.

    a = DateTime.parse('2014-11-22 12:35:05')
    #=> Sat, 22 Nov 2014 12:35:05 +0000
    a.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 12:35:05 +0000"
    a.in_time_zone.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 20:35:05 +0800"

    b = Time.zone.parse('2014-11-22 12:35:05')
    #=> 2014-11-22 12:35:05
    b.to_s(:rfc822)
    #=> "Sat, 22 Nov 2014 12:35:05 +0800"

Notice that `Time.zone.parse` has `+0800` timezone offset but `DateTime.parse`
has '+0000' timezone offset.

So `Time.zone.parse` may be better for you.

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
