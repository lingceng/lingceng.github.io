---
layout: post
title: "minitest mock and stud"
date: 2014-03-18 08:28
comments: true
categories: 
---

### capture io
Use `capture_io` to test output.

    {% codeblock lang:ruby %}
    out, err = capture_io do
      puts "Some info"
      warn "You did a bad thing"
    end

    assert_match %r%info%, out
    assert_match %r%bad%, err
    {% endcodeblock %} 

### mock
`capture_io` uses StringIO to wrap $stdout and $stderr.  
How to simulate user input or socket talks? Mock may do the job.

    {% codeblock stupidc.rb lang:ruby %}
    class Stupidc
      def initialize(input=STDIN, output=STDOUT)
        @input = input
        @output = output
      end

      def say_hello()
        @output.puts 'hello'
      end
    end
    {% endcodeblock %} 



    {% codeblock stupidc_spec.rb lang:ruby %}
    require 'minitest/autorun'

    describe Stupidc do
      before do
        @input = MiniTest::Mock.new
        @output = MiniTest::Mock.new

        @stupidc = Stupidc.new(@input, @output)
      end

     
      it "should copy file to source when file is target" do
        @output.expect :puts, nil, ['hello']
        @stupidc.say_hello()
        @output.verify
      end
    end
    {% endcodeblock %} 
    
### stub
Stub has similar function with mock, but no need to inject a property.   
Stub can simulate module methods or instance methods (They are the same in ruby, all class is the instance of Class) with ease.   
see [test](https://github.com/seattlerb/minitest/blob/master/test/minitest/test_minitest_mock.rb)

    {% codeblock lang:ruby %}
     def test_stub_yield_self
        obj = "foo"

        val = obj.stub :to_s, "bar" do |s|
          s.to_s
        end

        @tc.assert_equal "bar", val
      end
    {% endcodeblock %} 

