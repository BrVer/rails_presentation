# Ruby 2.1 INTERESTING PARTS

http://rkh.im/ruby-2.1


##Refinements
 - больше не эксперементальная фича, теперь их можно юзать в отдельном модуле, не изменяя скоуп всего файла:

```ruby
module Bar
  using Foo
  puts "bar".foo
end
```

но всё равно стоит подумать, надо ли их юзать


##Decimal Literals

для тех кому надоело видеть такое:

```
irb(main):001:0> 0.1 * 3
=> 0.30000000000000004
```

теперь можно делать вот так:

```ruby
irb(main):001:0> 0.1r
=> (1/10)
irb(main):002:0> 0.1r * 3
=> (3/10)
```

##Frozen String Literals

теперь можно содавать "замороженую" строку, объект которой будет создаваться только один раз

```ruby
class Foo
  def bar?(input)
    input == "bar"f
  end
end
```

##Required Keyword Arguments

```ruby
def foo(str:, delimiter: "-")
  str.split.join(delimiter)
end

foo(str: 'qwe asd', delimeter: '*') # qwe*asd
foo(delimeter: '*')                 # ArgumentError: missing keyword: str

```

##Method Definition returns Method Name

```
def foo() end # => :foo
```

вроде- бы НАФИГА?!

ан нет, теперь можно делать вот так например:

```ruby
# only foo and bar will be private
class Foo
  private def foo
  end

  private \
  def bar
  end

  def baz
  end
end
```

## StringScanner supports Named Captures

```ruby
require 'strscan'
s = StringScanner.new("foo")
s.scan(/(?<bar>.*)/)
puts s[:bar]
```

ruby 2.0:
```TypeError: no implicit conversion of Symbol into Integer```

ruby 2.1:
```foo```


##Faster Numbers for Serious Math
Ruby 2.1 is now faster when it comes to large numbers, as it uses 128 bit integers 
for representing bignums internally now if available. 
They also got an additional speed boost by using the GNU Multiple Precision Arithmetic Library.

##Ещё более быстрый GC за счёт нового супер-пупер механизма (RGenGC)


## .cause в эксепшенах

```ruby
class ExceptionalClass
  def exceptional_method
    cause = nil
    begin
      raise "Boom!"" # RuntimeError raised
    rescue => e
      raise StandardError, "Ka-pow!"
    end
  end
end
 
begin
  ExceptionalClass.new.exceptional_method
rescue Exception => e
  puts "Caught Exception: #{e.message} [#{e.class}]"
  puts "Caused by       : #{e.cause.message} [#{e.cause.class}]"
end
```

результат:
```
Caught Exception: Ka-pow! [StandardError]
Caused by       : Boom! [RuntimeError]
```