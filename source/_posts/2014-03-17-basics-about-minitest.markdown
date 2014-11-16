---
layout: post
title: "basics about minitest"
date: 2014-03-17 07:58
comments: true
categories:
---

+ home :: https://github.com/seattlerb/minitest
+ rdoc :: http://docs.seattlerb.org/minitest
+ vim  :: https://github.com/sunaku/vim-ruby-minitest

With Ruby 1.9, MiniTest entered standard lib.
MiniTest is pretty small and readable, here is all source file list:
> lib:
> hoe  minitest
>
> lib/hoe:
> minitest.rb
>
> lib/minitest:
> autorun.rb  benchmark.rb  hell.rb  mock.rb  parallel_each.rb  pride.rb  spec.rb  unit.rb


### Assert Methods
See  Minitest::Assertions doc or unit.rb source
    assert assert_equal assert_raises
    capture_io
    refute refute_empty
    ...

### Specs Expections
See Minitest::Expectations doc or spec.rb source
    must_be must_be_close_to
    wont_be_empty
    ...

Most methods just redirect to assert methods internally.
**describe** method is added to **Kernel** to start the DSL road;
expectations method is added to **Object**
    class Object
      include MiniTest::Expectations unless ENV["MT_NO_EXPECTATIONS"]
    end

### Simple Start

Given that you'd like to test the following class:
    class Meme
      def i_can_has_cheezburger?
        "OHAI!"
      end

      def will_it_blend?
        "YES!"
      end
    end

Unit tests
    require 'minitest/autorun'

    class TestMeme < MiniTest::Unit::TestCase
      def setup
        @meme = Meme.new
      end

      def test_that_kitty_can_eat
        assert_equal "OHAI!", @meme.i_can_has_cheezburger?
      end

      def test_that_it_will_not_blend
        refute_match /^no/i, @meme.will_it_blend?
      end

      def test_that_will_be_skipped
        skip "test this later"
      end
    end

Specs
    # the following have required in autorun
    # no need to repeat !
    # require 'minitest/unit'
    # require 'minitest/spec'
    # require 'minitest/mock'

    require 'minitest/autorun'

    describe Meme do
      before do
        @meme = Meme.new
      end

      describe "when asked about cheeseburgers" do
        it "must respond positively" do
          @meme.i_can_has_cheezburger?.must_equal "OHAI!"
        end
      end

      describe "when asked about blending possibilities" do
        it "won't say no" do
          @meme.will_it_blend?.wont_match /^no/i
        end
      end
    end



