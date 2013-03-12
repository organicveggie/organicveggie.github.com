---
layout: post
title: "Ruby mixin for the Enum pattern"
date: 2010-06-03 14:27
comments: true
categories: 
 - Design patterns
 - DRY
 - Programming
 - Rails
 - Ruby
---
Sometimes you just want to use an Enum. Unfortunately, if you're a Ruby developer, Ruby does not offer a native enum structure.
<!-- more -->
Here's a simple approach using a mixin module:

{% codeblock Enum.rb %}
module Enum
  def const_missing(key)
    @enum_hash[key]
  end
 
  def add_enum(key, value)
    @enum_hash ||= {}
    @enum_hash[key] = NameValuePair.new(value, key.to_s.downcase)
  end
 
  def each
    @enum_hash.each {|key, value| yield(key, value) }
  end
 
  def enums
    @enum_hash.keys
  end
 
  def enum_values
    @enum_hash.values
  end
 
  def get_enum_hash
    @enum_hash
  end
 
  def find_by_key(key)
    @enum_hash[key.upcase.to_sym]
  end
end
{% endcodeblock %}

The `Enum` mixin depends on a `NameValuePair` class to hold the data:

{% codeblock NameValuePair.rb %}
class NameValuePair
  attr_reader :label, :value
 
  def initialize(label, value)
    @label = label
    @value = value
  end
 
  def first
    @label
  end
 
  def last
    @value
  end
end
{% endcodeblock %}

I included first and last methods to better support the `select` and `options_for_select` helper methods in Rails. Here's how you might use it:

{% codeblock lang:ruby %}
class FooEnum
  extend Enum
 
   self.add_enum(:APPLE, "Apple")
   self.add_enum(:PEAR, "Pear")
   self.add_enum(:ALL, "All Fruit")
end
 
FooEnum::APPLE ==> #<NameValuePair @value="apple", @label="Apple">
FooEnum::ALL.value ==> "all"
FooEnum::ALL.label ==> "All Fruit"
FooEnum.find_by_key('apple') ==> #<NameValuePair @value="apple", @label="Apple">
{% endcodeblock %}