---
layout: post
title: "Migrate Legacy Data To Anthother Rails Project: Practical Tips About Using ActiveRecord"
date: 2016-03-18 20:38
comments: true
categories: ["rails", "sql"]
---

I migrated legacy data from one Rails project to its refactored Rails project
recently. I'll share some tips while I'm doing this job.

I fetched the legacy data directly from old DB and then save into new DB in a 
rake task. I used following code to connect the old DB:

    class DB < ActiveRecord::Base
      self.abstract_class = true
      if ENV['DEBUG']
        self.logger = Logger.new(STDOUT)
      end

      establish_connection adapter: 'mysql2', encoding: 'utf8',
        host: '127.0.0.1', port: '3306', database: 'databasename', 
        username: 'username'
    end

Then select result with raw SQL by using `select_all` method

    DB.connection.select_all("select * from users").each do |user|
      record = OpenStruct.new(record)
      new = UserInNewDB.find_or_initialize_by(id: record.id)
      user.attributes = record.to_h.slice(*%i[ phone name created_at updated_at])
      user.save! if user.changed?
    end

But it's hard while doing some complicated query with raw SQL. So I tried to
copy models in old project to new one. To keep new project clean, I mainly
reproduced the relations between models.

    class User < DB
      has_many :orders
    end

    class Order < DB
      belongs_to :user
    end

Then life gets better. I can do queries with ActiveRecord model and get all the
benefits.

    User.find_each do |record|
      # Do the migration
    end

## Tip0 Skip some callback

    Process.skip_callback(:save, :before, :log_changes)

## Tip1 Skip some validation

    if new.invalid? && 1 == new.errors.size && new.errors[:batch_id]
      new.save!(validate: false)
    else
      new.save!
    end

## Tip2 Cache basic table in a hash

    brands = Brand.all.index_by(&:name)
    Order.find_each do |record|
      new = NewOrder.find_or_initialize_by(id: record.id)
      new.brand = brands[name]
      new.save!
    end

## Tip3 Show current progress

    def show_process(name, total, index)
      printf "%s %.2f %%, %d / %d\r", name, index * 100.0 / total, index, total
    end

## Tip4 Find out records with no associated records

    Settlement.joins("left join orders on orders.settlement_id = settlements.id").
      where("orders.id is null")

## Tip5 Update column without triggering callbacks

    PayRecord.find(10059).update_column(:amount, 5986)

## Tip6 Do nested inner joins with ActiveRecord

    query = ProcessesChange.joins(process: [:technic, batch: :workgroup])

## Tip7 Use **squeel** to do outer join

    User.joins{recharges.outer}

## Tip8 Use `pluck` method to return array of data

    Order.joins(:item).group('items.brand').pluck("items.brand, count(orders.id) as order_count")

## Tip9 Use Mysql `GROUP_CONCAT` to return all items in a group

    sql =  Settlement.joins(:orders).group(:id).
      having("sum(orders.final_price) != sum(settlements.amount)").
      select("settlements.id, GROUP_CONCAT(orders.number),
      GROUP_CONCAT(orders.final_price), MIN(settlements.amount)").to_sql

    Settlement.connection.select_all(sql)

## Tip10 Manage rake task dependencies with an empty task

    task :user_module => [:users, :sources, :customers, :operators]
