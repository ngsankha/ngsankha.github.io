---
layout: post
title: "Ruby meta-hacking: Looking under the hood"
---

I have been programming in Ruby for quite sometime now and it is one of the most developer friendly languages I have used yet. Recently I was wondering how does the Ruby interpreter work internally and what are the data structures that make it tick.

### Ruby is a dynamically typed

Since Ruby is a dynamically typed language the variables have a lot of associated information to track what their type is. All this is abstracted out by Ruby to only give the necessary representation as needs to be seen by the programmer. What this essentially means is that if you have an integer that can be stored as a 32-bit integer, in Ruby due to all the extra information associated with the variable it would actually take up more memory than 4 bytes. This is in stark contrast with C where if I write `int32_t a = 5` it would take exactly 4 bytes of memory.

I wanted to see how Ruby internally stores these values and the header file [`ruby.h`](https://github.com/ruby/ruby/blob/trunk/include/ruby/ruby.h) proved instrumental in understanding the memory layout of Ruby types. Each Ruby object is represented in the interpreter as a `VALUE` type which is [type aliased](https://github.com/ruby/ruby/blob/787e878863d32d297373d7a4796271923246ae4d/include/ruby/ruby.h#L85) to an `unsigned long`. So it is a 64 bit number, and to use it we type cast it to a pointer of the object struct that we want to use, aka the memory address of the object that we are looking for.

At these memory locations we will find the actual data structures as defined in C in the source of the Ruby interpreter. All of these Ruby structs have a special member at start called `basic` which is of type struct `RBasic` as [defined](https://github.com/ruby/ruby/blob/787e878863d32d297373d7a4796271923246ae4d/include/ruby/ruby.h#L844-L847) in `ruby.h`.

<div class="shaky-container"><pre class="shaky">

                      --+
  +---------+           |
  |         |           |
  | flags <-+-VALUE     |
  |         |           |
  +---------+           +-- RBasic struct
  |         |           |
  | klass <-+-VALUE     |
  |         |           |
  +---------+           |
                      --+

</pre></div>

The struct `RBasic` defines 2 `VALUE` types in it:

* `flags`: multi-purpose flags that are used to register the struct types, etc.
* `klass`: this `VALUE` type is a pointer to a Ruby object which is the class of the object. This is the value that you would get if you called `my_object.class` in Ruby.

Let us take a look at how the Ruby `String` type [defined](https://github.com/ruby/ruby/blob/787e878863d32d297373d7a4796271923246ae4d/include/ruby/ruby.h#L951-L964) as the `RString` struct.

<div class="shaky-container"><pre class="shaky">

                                --+
  +------------+                  |
  |            |                  |
  |   basic  <-+-RBasic           |
  |            |                  |
  +------------+                  |
  | +--------+ |                  |
  | | heap <-+-+-struct           +-- RString struct
  | +--------+ |                  |
  | | ary  <-+-+-array            |
  | +--------+ |                  |
  |            |                  |
  |     as  <--+-union            |
  |            |                  |
  +------------+                  |
                                --+

</pre></div>

* `basic`: The same `RBasic` struct that is associated with this object.
* `as`: This is a union type, ie. it takes either of the two values:
  - `heap`: This struct stores the string length, the character bytes and a few other parameters.
  - `ary`: This is a simple array of characters with a null terminator.

This is pretty interesting because there are two ways in which the actual bytes are stored in the memory. If the string is more than a certain length (`RSTRING_EMBED_LEN_MAX + 1`) which is typically 24 bytes it is stored as simple byte array of length 24 bytes with all the remaining bytes null. Otherwise, it is [stored as a struct](https://github.com/ruby/ruby/blob/787e878863d32d297373d7a4796271923246ae4d/include/ruby/ruby.h#L954-L961), with the appropriate length and a pointer to the bytes that represent the string.

### Time for some meta-hacking

The next question that immediately comes to mind is that if this is how Ruby objects are represented in memory, can we use just our knowledge of Ruby to explore these properties and in the process just prove that we were right in understanding the data structures.

Turns out, yes it is possible to look under the hood of the Ruby interpreter - to explore each of these values of these structs. To do this we use the `fiddle` library to access the underlying C like values directly from Ruby.

{% highlight ruby %}
2.2.1 :001 > require 'fiddle'
 => true
2.2.1 :002 > str = "Test string!"
 => "Test string!"
{% endhighlight %}

In Ruby all objects have an unique object id. This isn't its memeory address but we can get the real memory address by multiplying the object id by 2. Note that this is really implementation dependent (in this case it works only in the default Ruby interpreter - the MRI). Once we have the address, we can create a pointer to that memory address easily with Fiddle.

{% highlight ruby %}
2.2.1 :003 > addr = str.object_id << 1
 => 140387096323280
2.2.1 :004 > ptr = Fiddle::Pointer.new(addr)
 => #<Fiddle::Pointer:0x007fae6950d000 ptr=0x007fae6b01e4d0 size=0 free=0x00000000000000>
{% endhighlight %}

Let us see if we can get the class of the string that we just defined from this pointer in memory. This value should present in the `klass` property of the `RBasic` struct. Looking at the diagrams above we can see that it is the second member of the `RBasic` struct. So we can get view of the struct in the memory and if we convert that to two VALUE type integers we will get the `flags` and `klass` members of the struct.

{% highlight ruby %}
2.2.1 :005 > r_basic = ptr[0, Fiddle::SIZEOF_LONG * 2]
 => "\x05\x00S\x00\x00\x00\x00\x00\xE8\xCE\x8Di\xAE\x7F\x00\x00"
2.2.1 :006 > flags, klass = r_basic.unpack("Q2")
 => [5439493, 140387071938280]
2.2.1 :007 > klass_ptr = Fiddle::Pointer.new(klass)
 => #<Fiddle::Pointer:0x007fae6953d6e0 ptr=0x007fae698dcee8 size=0 free=0x00000000000000>
2.2.1 :008 > klass_ptr.to_value
 => String
{% endhighlight %}

Thus by following the pointer from the `klass` variable and converting it to the Ruby object we see that we see the class of the string `str` we had defined. This is equivalent of calling `str.class`. This also shows how much complexity Ruby hides for us behind the scenes - all that pointer things are done and I am just returned a Ruby class by magic when I do `something.class`.

Getting the string value should also be easy. Since our defined string `Test string!` is smaller than 24 bytes, it would be stored as a fixed length array. Let's try to retrieve it.

{% highlight ruby %}
2.2.1 :009 > RSTRING_LEN_MAX = (Fiddle::SIZEOF_LONG * 3) / Fiddle::SIZEOF_CHAR
 => 24
2.2.1 :010 > str_data = ptr[Fiddle::SIZEOF_LONG * 2, RSTRING_LEN_MAX]
 => "Test string!\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
{% endhighlight %}

We can see the same string that we defined and all the other places in the array filled with the null character. So overall, this is what a Ruby string looks at the native level.

<div class="shaky-container"><pre class="shaky">

          +--------+
          | flags  |   +--> String
+------+  +--------+   |
| str *+->| klass *+---+
+------+  +--------+
  VALUE   |   as  *+------> "Test String!\0\0\0..."
          +--------+

</pre></div>

Now the next thing that comes to mind is if I change the value at any of the underlying memory locations the original Ruby value should also change. Let's try it.

{% highlight ruby %}
2.2.1 :011 > ptr[Fiddle::SIZEOF_LONG * 2] = "B".ord
 => 66
2.2.1 :012 > str
 => "Best string!"
{% endhighlight %}

The first character of the string was a `T`, whose memory location we just replaced with a `B`. Boom! The original string `str` changed as well. This probably the most powerful evidence that we playing with values stored in memory locations directly and it just updates our Ruby values as well. A bit care is neede when playing around with this because I managed to seg-fault Ruby quite a few times due to some malformed parameter.

### Taking further

A look at all the native Ruby structs defined in `ruby.h` gives an insight into how the internals work. For example `RArray` struct is also [defined](https://github.com/ruby/ruby/blob/787e878863d32d297373d7a4796271923246ae4d/include/ruby/ruby.h#L999-L1012) in a similar manner as a string. It also has a pointer to the heap and a simple array backed implementation containing the `VALUE` types. The array backed implmentation is used only when the array has less than or equal to 3 elements. Fiddle can be used to build C pointers to any of these Ruby types and we can then explore their underlying implementation from Ruby itself. Any change to an `VALUE` object is reflected throughout, for example in a Ruby array  there would be all Ruby objects. If we change any of the data pointed by the `VALUE` types, the array would also change.

{% highlight ruby %}
2.2.1 :013 > # declare an array
2.2.1 :014 > arr = ["hello", "world", "!"]
 => ["hello", "world", "!"]
2.2.1 :015 > arr_addr = arr.object_id << 1
 => 140526406752240

2.2.1 :016 > # get the pointer to the array
2.2.1 :017 > ptr = Fiddle::Pointer.new(arr_addr)
 => #<Fiddle::Pointer:0x007fced8f8ebb0 ptr=0x007fceda8e57f0 size=0 free=0x00000000000000>
2.2.1 :018 > RARRAY_LEN_MAX = (Fiddle::SIZEOF_LONG * 3) / Fiddle::SIZEOF_CHAR
 => 24

2.2.1 :019 > # data dump from the 3 element array memory
2.2.1 :020 > arr_value_data = ptr[Fiddle::SIZEOF_LONG*2, RARRAY_LEN_MAX]
 => "\x90X\x8E\xDA\xCE\x7F\x00\x00hX\x8E\xDA\xCE\x7F\x00\x00\x18X\x8E\xDA\xCE\x7F\x00\x00"

2.2.1 :021 > # get the 3 VALUE types of the 3 elements in the array
2.2.1 :022 > elems = arr_value_data.unpack("Q3")
 => [140526406752400, 140526406752360, 140526406752280]

2.2.1 :023 > # get the pointer to first element of the array
2.2.1 :024 > elem = Fiddle::Pointer.new(elems[0])
 => #<Fiddle::Pointer:0x007fcedb03e500 ptr=0x007fceda8e5890 size=0 free=0x00000000000000>

2.2.1 :025 > # see if we have the correct data
2.2.1 :026 > str_value_data = elem[Fiddle::SIZEOF_LONG*2, RSTRING_LEN_MAX]
 => "hello\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"

2.2.1 :027 > # replace 'h' with 'g'
2.2.1 :028 > elem[Fiddle::SIZEOF_LONG*2] = "g".ord
 => 103

2.2.1 :029 > # Yay! We have 'gello' instead of 'hello'
2.2.1 :030 > arr
 => ["gello", "world", "!"]
{% endhighlight %}

<div class="shaky-container"><pre class="shaky">

          +--------+
          | flags  |   +--> Array
+------+  +--------+   |
| arr *+->| klass *+---+
+------+  +--------+        +-------+-------+-------+
 VALUE    |   as  *+------> | VALUE | VALUE | VALUE*+-> "!"
          |        |        |   *   |   *   |       |
          +--------+        +---+---+---+---+-------+
                                |       |
                                v       +-----> "world"
                             "hello"

</pre></div>

### Unintended Consequences

Ruby is a dynamically typed and interpreted language. As we just saw the objects in Ruby are not just the bare "value" but also kept with all other supporting data and class of the object, etc. Every operation thus needs to check the type of the objects on the runtime and if the operation is a valid one create a new stuct in the memory - populate it with the correct `klass` and `flags`. Then the actual operation is done and the result is the stored in the new struct.

Also, this object model has a lot of pointer indirections making many operations slow. For example, if I am looping over an array when trying to print each of the values - it would lead to jumps to different regions in the memory to access each individual value. This is very different to a C array where all the values are laid out in a contiguous block of memory so looping operations are really really fast on it and at the same time saves memory.

### Conclusion

But I think this is a necessary price that we have to pay for a dynamically typed language. Even though under the hood we have all these pointer indirections - the language is really easy and forgiving and fast to develop in for the user, saving a lot of developer time and that is what makes Ruby a joy to use. We just used Ruby to hack Ruby object internals and this gives us way to explore how the interpreter works. This was an extremely fun exercise to look inside the language that I have been using for quite sometime!

<script type="text/javascript" src="/js/shaky.dart.js">
