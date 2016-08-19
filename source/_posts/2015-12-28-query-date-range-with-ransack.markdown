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

Certainly there's nothing between '2015-01-01 00:00' and '2015-01-01 00:00'.
But our users don't think so. They shout that there is a whole day from 2015-01-01 to 2015-01-01!

### Direct solution
OK. Users are gods.
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

The *created_at_lt* will convert to '2015-01-01 23:59'.

### DRY
I customed the ransack predicates to avoid duplication.
So I can just write the view like following:

<script src="https://gist.github.com/lingceng/65c58512d9bbb50799c7.js"></script>

### Maybe Another Way
Maybe we can change the js datepicker to set time to 59:59 by default.
I use [bootstrap-datetimepicker](http://eonasdan.github.io/bootstrap-datetimepicker/).
I find the javascript solution:

    = hidden_field_tag 'q[created_at_lteq]', params[:q].try(:[], :created_at_lteq)
    = date_field_tag '', params[:q].try(:[], :created_at_lteq).to_s.sub(/ .+/, ''), onchange: "$(this).prev().val($(this).val() != '' ? $(this).val() + ' 23:59:59' : '')"

