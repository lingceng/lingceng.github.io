---
layout: post
title: "Tips About Using AASM gem"
date: 2015-12-19 12:26
comments: true
categories: [rails]
---

Recently, I'm using [aasm gem](https://github.com/aasm/aasm) to do order production status control.
Here I have some tips about using aasm gem.

I'm using aasm 4.5.0 and rails 4.2.0. First, I post a snippet here for better explaination.

{% codeblock order.rb lang:ruby %}
class Order
  include AASM

  aasm column: :production_status do
    state :idle, initial: true
    state :at_center
    state :ready

    after_all_transitions :_save_production_status_change

    event :check_in, before: :set_current_step_to_center, guard: :_status_fit do
      transitions from: [:idle], to: :at_center, after: :_change_status_to_repairing
    end

    event :to_ready, before: :set_current_step, guard: :_status_fit do
      transitions from: [:at_center, :idle], to: :ready, after: :_change_status_to_repairing
    end
    ...
  end

  ...
end
{% endcodeblock %}

## How to keep actions in transaction?
Of course, you can acheive it by wrapping actions in a transaction. eg.

    Order.transaction do
      order.check_in
      order.save
    end

However, aasm also provide a bang method to acheive it. eg.

    order.check_in!

But the two forms is not the same.

Please take care the save point if you choose to use `order.check_in!`.
`order.check_in!` internally calls save after `new_state enter` callback.
So, changes in callbacks after `new_state enter`  will not auto-save.

See the list of callbacks.

    begin
      event           before
      event           guards
      transition      guards
      old_state       before_exit
      old_state       exit
                      after_all_transitions
      transition      after
      new_state       before_enter
      new_state       enter
      ...update state...               # here is the save point
      event         success            # if persist successful
      old_state       after_exit
      new_state       after_enter
      event           after
    rescue
      event           error
    end

Let's see a example. If I wrote the `check_in` event like following:

    event :check_in, before: :set_current_step_to_center, guard: :_status_fit,
      after: :_change_status_to_repairing do
      transitions from: [:idle], to: :at_center
    end

    def _change_status_to_repairing
      status = 'repairing'
    end

The `_change_status_to_repairing` is at `event after` callback.
So, when I run `order.check_in!`, `order.status` will not persist.

    order.check_in!
    order.reload.status == 'repairing'
    # => false

Of cause, you can call save in `_change_status_to_repairing`
But the order will be persisted even when you run non-bang method
`order.check_in`.

    def _change_status_to_repairing
      status = 'repairing'
      save
    end

Oh. Maybe you should choose to use the first form.

## Avoid using Proc.new with optional argument in guards and callbacks
See https://github.com/aasm/aasm/issues/293

## Take care of before event filter
See https://github.com/aasm/aasm/issues/294

## duplicate global callbacks caused by after_all_transitions
See https://github.com/aasm/aasm/issues/297
