---
date: "2019-08-27T02:14:18.124Z"
title: crystal简介
---

关注 crystal 也有一段时间了，看到多线程的 pr 已经提了，今天简单写一下。

> Fast as C, Slick as Ruby

# 语法

crystal 的语法和 Ruby 比较类似。

```ruby
# A very basic HTTP server
require "http/server"

server = HTTP::Server.new do |context|
  context.response.content_type = "text/plain"
  context.response.print "Hello world, got #{context.request.path}!"
end

puts "Listening on http://127.0.0.1:8080"
server.listen(8080)
```

# 类型系统

crystal 的一大卖点就是静态类型系统，但是写起来又和脚本语言类似。

```ruby
def shout(x)
  # Notice that both Int32 and String respond_to `to_s`
  x.to_s.upcase
end

foo = ENV["FOO"]? || 10

typeof(foo) # => (Int32 | String)
typeof(shout(foo)) # => String
```

# 空引用检查

crystal 可以在编译的时候检查空引用，避免出现空指针异常。

```ruby
if rand(2) > 0
  my_string = "hello world"
end

puts my_string.upcase
```

如果运行上述的代码，执行结果如下：

```shell
$ crystal hello_world.cr
Error in hello_world.cr:5: undefined method 'upcase' for Nil (compile-time type is (String | Nil))

puts my_string.upcase
```

# 宏

另一个重要的特性是宏。通过宏，可以实现向 ruby 那么强大的元编程。

```ruby
class Object
  def has_instance_var?(name) : Bool
    {{ @type.instance_vars.map &.name.stringify }}.includes? name
  end
end

person = Person.new "John", 30
person.has_instance_var?("name") #=> true
person.has_instance_var?("birthday") #=> false
```

# 并发

crystal 的并发是通过绿色线程实现的，即 fibers。和 Go 的并发模式很像，也是基于 channel 的 CSP 模型。

```ruby
channel = Channel(Int32).new
total_lines = 0
files = Dir.glob("*.txt")

files.each do |f|
  spawn do
    lines = File.read(f).lines.size
    channel.send lines
  end
end

files.size.times do
  total_lines += channel.receive
end

puts total_lines
```

# C 绑定

C 语言一般用来实现比较底层的系统，而且 C 的生态丰富，一般现代语言都会提供 C 绑定，来复用 C 的生态。

```ruby
# Fragment of the BigInt implementation that uses GMP
@[Link("gmp")]
lib LibGMP
  alias Int = LibC::Int
  alias ULong = LibC::ULong

  struct MPZ
    _mp_alloc : Int32
    _mp_size : Int32
    _mp_d : ULong*
  end

  fun init_set_str = __gmpz_init_set_str(rop : MPZ*, str : UInt8*, base : Int) : Int
  fun cmp = __gmpz_cmp(op1 : MPZ*, op2 : MPZ*) : Int
end

struct BigInt < Int
  def initialize(str : String, base = 10)
    err = LibGMP.init_set_str(out @mpz, str, base)
    raise ArgumentError.new("invalid BigInt: #{str}") if err == -1
  end

  def <=>(other : BigInt)
    LibGMP.cmp(mpz, other)
  end
end
```

# 依赖管理

任何一个偏工程性的语言，都会提供一个包管理系统。crystal 的包管理是 shards，其实和 go module 类似。这种项目级别的包管理其实更为实用一些。

但是 go 的任何一个项目，其实都可以是一个包，crystal 还是会有一些限制的。

```ruby
name: my-project
version: 0.1
license: MIT

crystal: 0.21.0

dependencies:
  mysql:
    github: crystal-lang/crystal-mysql
    version: ~> 0.3.1
```
