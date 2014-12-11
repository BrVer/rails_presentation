# Ruby 2.0 INTERESTING PARTS

http://globaldev.co.uk/2013/03/ruby-2-0-0-in-detail/
http://inet777.ru/news/8407/vyipuschen-reliz-ruby-2-0-0

##Новшества языка, ядро:

###именованые аргументы методов (keyword arguments), которые придают дополнительную гибкость API

```ruby
def wrap(string, before: "<", after: ">")
  # no need to retrieve options from a hash
  "#{before}#{string}#{after}"
end
 
# optional
p wrap("foo")               #=> "<foo>"
 
# one or the other
p wrap("foo", before: "#<") #=> "#<foo>"
p wrap("foo", after: "]")   #=> "<foo]"
 
# order not important
p wrap("foo", after: "]", before: "[") #=> "[foo]"
 
# double splat to capture all keyword arguments,
# or use as hash as keyword arguments
def capture(**opts)
  opts
end
p capture(foo: "bar") #=> {:foo=>"bar"}
 
# keys must be symbols
opts = {:before => "(", :after => ")"}
p wrap("foo", opts) #=> "(foo)"
 
# the old hash style syantax is still accepted for keyword arguments
p wrap("foo", :before => "{", :after => "}") #=> "{foo}"
```

###Module#prepend - новый способ расширения классов
С помощью <code>prepend</code> методы модуля устанавливаются первоочередными при поиске метода в классе, 
в который подключен модуль.

Модуль, подключаемый с помощью <code>prepend</code> может вызвать метод, который он перекрывает 
в классе с помощью обращения <code>super</code>:

inheritance chain:

Object<br/>
superclass<br/>
included module<br/>
class<br/>
prepended module<br/>

```ruby
module A
  def m
    puts "A"
    super
  end
end
 
class B
 
  include A
 
  def m
    puts "B"
  end
end
 
class C
 
  prepend A
 
  def m
    puts "C"
  end
end
 
 
B.new.m #=> B
 
C.new.m #=> A
        #=> C
```

###Refinements

Refinements is a neat idea, but the original implementation came with a weird edge cases and possible performance 
penalties, so what we get with Ruby 2.0.0 is a rather scaled back, and slightly less useful version of the original.

You create a refinement to a class, name spaced inside a module

```ruby
module NumberQuery
  refine String do
    def number?
      match(/\A[1-9][0-9]*\z/) ? true : false
    end
  end
end
```

this refinement isn’t visible by default

```ruby
"123".respond_to?(:number?)   #=> false
```

once you declare that you are using the module with the refinement, then it becomes visible.

```ruby
using NumberQuery
"123".number?                 #=> true
```

###литералы %i и %I, которые позволяют легко создавать массивы обозначений (symbols):

```ruby
%i{hurray huzzah whoop} #=> [:hurray, :huzzah, :whoop]

%I{#{1 + 1}x #{2 + 2}x}   #=> [:"2x", :"4x"]
```

###__dir__ - возвращает каталог текущего исполняющегося файла
 (что-то типа ```File.dirname(File.realpath(__FILE__))```)
 
###Unbound methods from a module can be bound to any object

This one might sound like gibberish, or some minor change to something you’ll never use, 
but it’s actually a really great new feature.

You can get ahold of a method object from any class or module, with instance_method

```ruby
Object.instance_method(:to_s)   #=> #<UnboundMethod: Object(Kernel)#to_s>
```

However, this method isn’t bound to anything, it has no self, and can’t be called. 
To call it you have to bind the method to an object, but methods have to be bound to an 
object of the same class as the one the method was taken from.

But now we can take a method from a module and bind it to any object

```ruby
module Bar
  def bar
    "bar"
  end
  def baz
    "baz"
  end
end

Bar.instance_method(:bar).bind(Object.new).call   #=> "bar"
```
 
This means define_method also accepts unbound methods for modules, which will let us implement a selective include

```ruby
module Kernel
  def from(mod, include: [])
    raise TypeError, "argument must be a module" unless Module === mod
    include.each do |name, original|
      define_method(name, mod.instance_method(original || name))
    end
  end
end

class Foo
  from Bar, include: {:qux => :bar}
end

f = Foo.new
f.qux                 #=> "bar
f.respond_to?(:baz)   #=> false
```

###кодировка UTF-8 по-умолчанию


##Библиотеки:

###Enumerable#lazy and Enumerator::Lazy - "Ленивые" итераторы.
   
```ruby
lines = File.foreach('a_very_large_file')
            .lazy # so we only read the necessary parts!
            .select {|line| line.length < 10 }
            .map(&:chomp)
            .each_slice(3)
            .map {|lines| lines.join(';').downcase }
            .take_while {|line| line.length > 20 }
 
# => Lazy enumerator, nothing executed yet
lines.first(3) # => Reads the file until it returns 3 elements
               # or until an element of length <= 20 is
               # returned (because of the take_while)
 
# To consume the enumerable:
lines.to_a  # or...
lines.force # => Reads the file and returns an array
lines.each{|elem| puts elem } # => Reads the file 
                              # and prints the resulting elements
```
   
###Enumerator#size and Range#size, для "ленивого" вычисления размера

###to_h - новый метод, новое соглашение преобразования в Hash

```ruby
{:foo => "bar"}.to_h               #=> {:foo=>"bar"}
nil.to_h                           #=> {}
Struct.new(:foo).new("bar").to_h   #=> {:foo=>"bar"}
require "ostruct"
open = OpenStruct.new
open.foo = "bar"
open.to_h                          #=> {:foo=>"bar"}
```

##новый движок регулярных выражений regexp - Onigmo (a fork of Oniguruma). 
This is a fork of the Oniguruma regexp engine used by 1.9, with a few more features. 
The new features seem Perl-inspired, with a good reference.

##Rubygems Gemfile support

Rubygems can now use your Gemfile (or Isolate, or gem.deps.rb) to install gems and load activation information.

You can install the gems listed in your Gemfile (and their dependancies) by specifying the --file (or -g) option. 
This only uses the Gemfile, not Gemfile.lock, so only versions specified in the Gemfile will be respected.

```gem install --file Gemfile```

 - не очень интересно, потомучто всё равно юзаем ```bundle install``` 


###RDoc markdown support

RDoc now understands markdown, to run rdoc with markdown formatting set the markup option

```rdoc --markup markdown```
This can be saved in your project with a .doc_options file so you don’t need to repeat it every time

```rdoc --markup markdown --write-options```

###TracePoint

TracePoint is a new object oriented version of the old Kernel#set_trace_func. 
It allows you to trace the execution of your Ruby code, 
this can be really handy when debugging code that isn’t terribly straight forward.


###Asynchronous Thread interrupt handling

Ruby threads can be killed or have an exception raised in them by another. 
This isn’t a terribly safe feature as the killing thread doesn’t know what the thread being killed is doing, 
you could end up killing a thread in the middle of some important resource allocation or deallocation. 
We now have a feature to deal with this more safely.

As an example, the stdlib timeout library works by spawning a second thread, 
which waits for the specified amount of time, and then raises and exception in the original thread.

```ruby
Thread.handle_interrupt(Timeout::ExitException => :immediate)
```



##Отладчик:

-  поддержка отладчика
-  поддержка DTrace, который позволяет диагностику в production
-  TracePoint - является улучшенным API трассировки


##Производительность:

-  улучшенная производительность
-  GC optimization by bitmap marking
-  оптимизация Kernel#require которая делает загрузку Rails-приложений очень быстрой
-  оптимизированная VM (Virtual Machine) - улучшена диспечерезация методов (method dispatch)
-  оптимизированы операции с плавающей точкой
-  Создатели Ruby сообщают о том, что вводится Flonum и наибольшего прироста производительности можно будет добиться на 64-разрядных платформах и операционных системах.

Кроме того,  2.0.0 включает Refinements, которая добавляет новое понятие к модульному принципу Ruby.

Совместимость с предыдущими версиями

Разработчики также позаботились, чтобы сделать 2.0.0 совместимым с 1.9. Будет гораздо проще перейти с 1.9 на 2.0, чем это было с 1.8 на 1.9.

