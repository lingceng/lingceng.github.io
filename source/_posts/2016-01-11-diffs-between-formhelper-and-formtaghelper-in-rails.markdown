---
layout: post
title: "Diffs Between FormHelper and FormTagHelper in Rails"
date: 2016-01-11 13:10
comments: true
categories: ["Rails"]
---

We can use **form_for** or **form_tag** to build forms in Rails.
But their parameters act differently, which always bites me.

For example, I have to wrap html options into a 'html' key when I change from
form_tag to form_for.

    = form_tag '/techloglogies/board', method: :get, class: 'form-inline'

    = form_for @search, url: '/techloglogies/board', method: :get, html: { class: 'form-inline' }

And as the 3rd parameter of `f.select`  is options than html_options.
So I have to add an empty hash `{}` there to skip it.

    = select_tag 'step',
      options_for_select(Order::PURE_STEPS.invert, params[:step]),
      class: 'form-control input-sm', style: 'width: 100px'

    = f.select :step, Order::PURE_STEPS.invert, {},
      class: 'form-control input-sm', style: 'width: 100px'

### Why they acts differently? Isn't f.select built on select_tag?
**The short answer is NO.**
f.select is not built on select_tag.

Let's dig into the source.

#### Here's where form_for defined.

    module ActionView::Helper::FormHelper

      def text_field(object_name, method, options = {})
        Tags::TextField.new(object_name, method, self, options).render
      end

      def form_for(...)
        ...
      end
    end

In FormHelper above also defined a class named ActionView::Helpers::FormBuilder.
The form_for uses the FormBuilder.

Most methods in FormHelper are be delegated by FormBuilder.

    # in FormBuilder
    def #{selector}(method, options = {})  # def text_field(method, options = {})
      @template.send(                      #   @template.send(
        #{selector.inspect},               #     "text_field",
        @object_name,                      #     @object_name,
        method,                            #     method,
        objectify_options(options))        #     objectify_options(options))
    end                                    # end

Unlike text_field, select helper defined in ActionView::Helpers::FormOptionsHelper.
And a select method in FormBuilder delegates the FormOptionsHelper#select.

text_field or select finnally use classes under module ActionView::Helpers::Tags

#### FormTag and SelectTag are defined in ActionView::Helper::FormTagHelper
**ActionView::Helper::FormTagHelper seems totally another implement.**
Most tags helper are component directly in the methods. eg.

    def text_field_tag(name, value = nil, options = {})
      tag :input, { "type" => "text", "name" => name, "id" => sanitize_to_id(name), "value" => value }.update(options.stringify_keys)
    end

select_tag and FormBuilder#select do dot shere the implement.
They implement the include_blank and other options separately.
Is it a historical reason?

#### How to avoid converts?
[Use form object](https://www.reinteractive.net/posts/158-form-objects-in-rails) to always use form_for style.


