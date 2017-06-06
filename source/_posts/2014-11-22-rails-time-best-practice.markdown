---
layout: post
title: "Rails Time Best Practice"
date: 2014-11-22 18:38
comments: true
categories: [rails, ruby]
---

### Setup

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

`config.active_record.default_timezone` determines whether to use Time.zone.local
 (if set to :local) or Time.zone.utc (if set to :utc) when pulling dates and times
from the database.  The default is :utc.

**Update:**  
Let active_record.default_timezone be :utc is a better practice.  
Always save the utc to database.  
It helps when you make a world-wide application.

### Rails console datetime ouptut

Run following codes in rails console.

    l = FinanceItem.create
    #=> #<FinanceItem id: 1, created_at: "2014-11-22 03:00:32", updated_at: "2014-11-22 03:00:32" >
    l.created_at
    #=> Sat, 22 Nov 2014 11:00:32 CST +08:00

The rails console calls `inspect` to show return value.  
The default datatime inspcet is to_s(:db).  
So the created_at is "2014-11-22 03:00:32".

But `l.created_at`is an instance of ActiveSupport::TimeWithZone.
So it's output is "Sat, 22 Nov 2014 11:00:32 CST +08:00"

### Time.zone.parse VS Datetime.parse
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

So `Time.zone.parse` may be better for you.

### Time.now VS Time.current
Let's see the definition of `Time.current`

    # File activesupport/lib/active_support/core_ext/time/calculations.rb, line 29
    def current
      ::Time.zone ? ::Time.zone.now : ::Time.now
    end

Time.now uses the **system time zone** because it's is part of the Ruby standard library.  
Time.zone.now will set zone with `config.time_zone`.

Using `Time.now` make troubles when your system time zone is different with `config.time_zone`

### Best practice
Set the following config.

    config.active_record.default_timezone = :utc
    config.time_zone = 'YourLocalName'

Use `Time.zone.parse` and do **NOT** use `DateTime.parse`.  
Use `Time.zone.now` or `Time.current` and do **NOT** use `Time.now`.  
Thus we can keep all time class to TimeWithZone and get consistent behavior.
