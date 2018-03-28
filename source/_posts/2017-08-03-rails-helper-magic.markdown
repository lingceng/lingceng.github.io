---
layout: post
title: "Rails Helper Magic"
date: 2017-08-03 21:00 
comments: true
categories: ["rails", "helper", "view"]
---

### Rails view helpers default value magic
Following will set default value for *:first_name* field if an instance variable @person present.

    <%= form_for :person do |f| %>
      First name: <%= f.text_field :first_name %><br />
    <% end %>

Actually the following will set the default value too.

    First name: <%= text_field :person, :first_name %><br />

The magic is hidden in *retrieve_object* method in *lib/action_view/helpers/tags/base.rb*.
The @template_object is the view context here.

    def retrieve_object(object)
      if object
        object
      elsif @template_object.instance_variable_defined?("@#{@object_name}")
        @template_object.instance_variable_get("@#{@object_name}")
      end
      ...
    end

Most form helpers such as *text_field*, *check_box* and *select*, is based on *Helpers:Tags::Base*.
So the magic works for them too.

### Rails helpers has two similar definitions
There are always two similar definitions for most view helpers.
Such as *check_box* and *select*.

One is in **ActionView::Helpers::[Some]Helper**.

    check_box(object_name, method, options = {}, checked_value = "1", unchecked_value = "0")

The other is in **ActionView::Helpers::FormBuilder**.

    check_box(method, options = {}, checked_value = "1", unchecked_value = "0")

The difference is the first definition needs to specify the **object_name**.

The truth is most helpers provide a wrapper for **FormBuilder** class.

See the **check_box** in **FormBuilder** below.
The **@template** is the view context here.
The **objectify_options** will pass in the current object of the form.

    def check_box(method, options = {}, checked_value = "1", unchecked_value = "0")
      @template.check_box(@object_name, method, objectify_options(options), checked_value, unchecked_value)
    end

By the way, helpers ends with 'tag', such as **check_box_tag** and **select_tag**, have another implement.  They have much different usage.

### Difference between form_for and fields_for
Both are helpers and very similar.
**form_for** handles more options for URL and methods.

    def form_for(record, options = {}, &block)
      ...
      builder = instantiate_builder(object_name, object, options)
      output  = capture(builder, &block)
      form_tag(options[:url] || {}, html_options) { output }
    end

    def fields_for(record_name, record_object = nil, options = {}, &block)
      builder = instantiate_builder(record_name, record_object, options)
      capture(builder, &block)
    end

But **fields_for** also defined in FormBuilder class while **form_for** did not.
So we can use fields_for as following:

    f.fields_for :orders do |order_form|
    end

The FormBuilder#fields_for handle some details about **accepts_nested_attributes_for**.
