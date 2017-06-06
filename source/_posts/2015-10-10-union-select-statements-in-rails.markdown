---
layout: post
title: "Union select statements in Rails"
date: 2015-10-10 11:42
comments: true
categories: [rails, sql]
---
Here we have a table named `user_statuses` like following

     name    | status
     --------+-------
      'jack' |   1
      'jack' |   2
      'sam'  |   6
      'lucy' |   1
      'toy'  |   2
      'toy'  |   7
      'jack' |   7

I want to collect the status with follwing rule:

+ Keep all [1-2] statuses
+ Ignore other status above 2 if one has any [1-2] status
+ keep only one ohter status above 2 if one do not has any [1-2] status

So the result should like following:

     name    | status
     --------+-------
      'jack' |   1
      'jack' |   2
      'sam'  |   6
      'lucy' |   1
      'toy'  |   2

The real life case is more complicated. I want to achieve this rule in SQL.
As a result I can do some group or something else later.

After some searches, I found out `union all` can do the job.
But call `union` on ActiveRelation returns a Arel::Nodes::Union but ActiveRelation.
Gem [active record union](https://github.com/brianhempel/active_record_union) get a
good solution.

Here's my demo sulution:

      statuses = [1, 2]
      left = UserStatus.where(status: statuses).select(:name, :status)
      right = UserStatus.select(:name, 'MIN(status) as status').
        group(:name).
        having('MIN(status) > 2')
      query = left.union_all(right)

