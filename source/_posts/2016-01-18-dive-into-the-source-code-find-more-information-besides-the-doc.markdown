---
layout: post
title: "Dive Into The Source Code: Find More Information Besides The Doc"
date: 2016-01-18 21:27
comments: true
categories: ["rails"]
---

We can use **pry** and  **pry-byebug** gem to help us dive into the source code.

### Let's see an example about `to_json` vs `as_json`
I found there's a `to_json` and `as_json` method for active record query result.
`to_json` returns String of json.
`as_json` returns a hash representing the model.

{% codeblock lang:ruby %}
> Order.where(number: "201504211405490").to_json
# => "[{\"production_status\":\"idle\",\"id\":11,\"number\":\"201504211405490\"}]"
> Order.where(number: "201504211405490").to_json.class
# => String

> Order.where(number: "201504211405490").as_json
# => [{"production_status"=>"idle", "id"=>11, "number"=>"201504211405490"}]
> Order.where(number: "201504211405490").as_json.class
# => Hash
{% endcodeblock ruby %}

I find that `as_json` can accepts some optons.

{% codeblock lang:ruby %}
> Order.where(number: "201504211405490").as_json(only: :number)
# => [{"number"=>"201504211405490"}]
{% endcodeblock ruby%}

But one day I found somebody used similar options with `to_json` method.
I searched the doc. No doc says `to_json` can do things like following:

{% codeblock lang:ruby %}
> Order.where(number: "201504211405490").to_json(only: :number)
# => "[{\"number\":\"201504211405490\"}]"
{% endcodeblock ruby%}

But the code above actually works.
Is `to_json` uses `as_json` under the hood?

### Dive into the source
Now we can check the source code to find the answer.

{% codeblock lang:ruby %}
> $ Order.where(number: "201504211405490").to_json

From: /Users/lingceng/.rvm/gems/ruby-2.2.0@baozheng/gems/activesupport-4.2.0/lib/active_support/core_ext/object/json.rb @ line 31:
Owner: Object
Visibility: public
Number of lines: 9

def to_json_with_active_support_encoder(options = nil)
  if options.is_a?(::JSON::State)
    # Called from JSON.{generate,dump}, forward it to JSON gem's to_json
    self.to_json_without_active_support_encoder(options)
  else
    # to_json is being invoked directly, use ActiveSupport's encoder
    ActiveSupport::JSON.encode(self, options)
  end
end
{% endcodeblock ruby%}

Then we edit the method and add a **binding.pry** before the `if options.is_a?(::JSON::State)` line.
**Remember to recover the change after debug.**

    > edit Order.where(number: "201504211405490").to_json

Then run code again:

    > Order.where(number: "201504211405490").to_json

Now we'll get into `to_json_with_active_support_encoder` method and start to debug.
After some **next** and **step** comand.
We can find that `to_json` uses ActiveSupport::JSON.encode, which uses `as_json`.

    # /lib/active_support/json/encoding.rb#34
    # Encode the given object into a JSON string
    def encode(value)
      stringify jsonify value.as_json(options.dup)
    end

Yes! `to_json` uses `as_json`. We find the answer. So we can use to_json(only: :number) with
confidence.

### Conclusion
Pry and pry-byebug provides many useful commands.
Using these commands can help to find the definition of a method, track the call stacks, understand the source structure.

This is beauty of open source.
Do not limit yourself.
Dive into the code and get your information.

