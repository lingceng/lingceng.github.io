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

#### Test datetime output

Run with `rails console`.

    pry(main)> l = FinanceItem.create
    => #<FinanceItem id: 1, created_at: "2014-11-22 03:00:32",
        updated_at: "2014-11-22 03:00:32" >
    pry(main)> l.created_at
    => Sat, 22 Nov 2014 11:00:32 CST +08:00
    pry(main)> l.created_at.to_s(:db)
    => "2014-11-22 03:00:32"

#### Datetime output is different

Here is the reason.

    TimeWithZone to_s(:db)
    :db format outputs time in UTC; all others output time in local

#### `default_timezone` is so necessary

`config.time_zone` sets the default time zone for the application
and enables time zone awareness for Active Record.

`config.active_record.default_timezone` determines whether to use Time.local
 (if set to :local) or Time.utc (if set to :utc) when pulling dates and times
from the database.
The default is :utc.

`utc()`
Returns a Time or DateTime instance that represents the time in UTC.

`local()`
Method for creating new ActiveSupport::TimeWithZone instance
in time zone of self from given values.

    Time.zone = 'Hawaii'                    # => "Hawaii"
    Time.zone.local(2007, 2, 1, 15, 30, 45) # => Thu, 01 Feb 2007 15:30:45 HST -10:00

#### Take care when parse date string

Let's run the test

    pry(main)> a = DateTime.parse('2014-11-22 12:35:05')
    => Sat, 22 Nov 2014 12:35:05 +0000
    pry(main)> a.to_s(:rfc822)
    => "Sat, 22 Nov 2014 12:35:05 +0000"
    pry(main)> a.in_time_zone.to_s(:rfc822)
    => "Sat, 22 Nov 2014 20:35:05 +0800"

    pry(main)> b = Time.zone.parse('2014-11-22 12:35:05')
    => 2014-11-22 12:35:05
    pry(main)> b.to_s(:rfc822)
    => "Sat, 22 Nov 2014 12:35:05 +0800"
    pry(main)> c = '2014-11-22 12:35:05'.to_time
    => 2014-11-22 12:35:05
    pry(main)> c.to_s
    => "2014-11-22 12:35:05 +0800"

Notice that `Time.zone.parse` has `+0800` timezone offset.

### Conclusion

Always set following options.

    active_record.default_timezone = :local
    time_zone = 'Beijing'

Use `Time.zone.parse` do **not** use `DateTime.parse`.
