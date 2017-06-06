---
layout: post
title: "Minitest Mock And Stud"
date: 2014-03-18 08:28
comments: true
categories: [ruby, test]
---

### Capture IO

Use `capture_io` to test standard output.  
`capture_io` uses StringIO to wrap $stdout and $stderr.

    out, err = capture_io do
      puts "Some info"
      warn "You did a bad thing"
    end

    assert_match /info/, out
    assert_match /bad/, err

### Mock

    class Stupidc
      def initialize(input=STDIN, output=STDOUT)
        @input = input
        @output = output
      end

      def say_hello()
        @output.puts 'hello'
      end
    end


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

### Stub
see [test](https://github.com/seattlerb/minitest/blob/master/test/minitest/test_minitest_mock.rb)

    def test_stub_yield_self
      obj = "foo"

      val = obj.stub :to_s, "bar" do |s|
        s.to_s
      end

      @tc.assert_equal "bar", val
    end

