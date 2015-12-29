---
layout: post
title: "Query Date Range with Ransack"
date: 2015-12-28 19:42
comments: true
categories: ["Rails", "Ransack"]
---

### The Traditional Way
Here I have a table of change records in my rails app.
And I have added a query for *created_at* with [ransack](https://github.com/activerecord-hackery/ransack).


{% codeblock app/controllers/production_status_changes_controller.rb lang:ruby %}
class ProductionStatusChangesController < PlainController
  def index
    @q = ProductionStatusChange.ransack(params[:q])
    @orders = @q.result.includes(:order).page(params[:page]).per(params[:per])
  end
end
{% endcodeblock %}

{% codeblock app/views/production_status_changes/index.html.erb lang:erb %}
<%= search_form_for @q, url: production_status_changes_path, class: 'form-inline' do |f| %>
  <%=  f.label 'Create At' %>
  <%= f.search_field :created_at_gteq, class: 'form-control input-sm', 'datepicker' => true %>
  <%= f.search_field :created_at_lt, class: 'form-control input-sm', 'datepicker' => true %>
<% end %>
{% endcodeblock %}

### The Problem
Everything works fine until users start to use it.
They are surpised that, when query with "2015-01-01" and "2015-01-01", nothing comes out.

I of couse know that nothing exists between '2015-01-01 00:00' and '2015-01-01 00:00'.
But our users don't think so. They shout that there is a whole day form 2015-01-01 to 2015-01-01!

### Direct solution
Ok, users are gods.
So I add some codes in my controller:

{% codeblock app/controllers/production_status_changes_controller.rb lang:ruby %}
def index
  params[:q] ||= {}
  if params[:q][:created_at_lt].present?
    params[:q][:created_at_lt] = params[:q][:created_at_lt].to_date.end_of_day
  end
  @q = ProductionStatusChange.ransack(params[:q])
  @orders = @q.result.includes(:order).page(params[:page]).per(params[:per])
end
{% endcodeblock %}

The *created_at_lt* will convert to '2015-01-01 59:59'.

### DRY
I customed the ransack predicates to avoid duplication.

{% codeblock config/initializers/ransack.rb lang:ruby %}
# Custom ransack predicate to simplify query for date range
#
# See lib/ransack.rb in ransack gem
# See wiki https://github.com/activerecord-hackery/ransack/wiki/Custom-Predicates
Ransack.configure do |config|

  config.add_predicate 'beginning_of_day_gteq',
    arel_predicate: 'gteq',
    formatter: proc { |v| v.to_date.beginning_of_day },
    validator: proc { |v| v.present? },
    type: :string

  config.add_predicate 'end_of_day_lt',
    arel_predicate: 'lt',
    formatter: proc { |v| v.to_date.end_of_day },
    validator: proc { |v| v.present? },
    type: :string
end
{% endcodeblock %}

So I can just write the view like following:

{% codeblock app/views/production_status_changes/index.html.erb lang:erb %}
<%= search_form_for @q, url: production_status_changes_path, class: 'form-inline' do |f| %>
  <%=  f.label 'Create At' %>
  <%= f.search_field :created_at_beginning_of_day, class: 'form-control input-sm', 'datepicker' => true %>
  <%= f.search_field :created_at_end_of_day_lt, class: 'form-control input-sm',  'datepicker' => true %>
<% end %>
{% endcodeblock %}

### Maybe Another Way
Maybe we can change the js datepicker to set time to 59:59 by default.
I use [bootstrap-datetimepicker](http://eonasdan.github.io/bootstrap-datetimepicker/).
Maybe I will find it out later or someone else can help me?


