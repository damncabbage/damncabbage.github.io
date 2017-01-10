---
layout: post
title: "Flags vs Constants, and Short-Long Days of the Week"
date: 2011-12-12 22:52
comments: true
categories:
- PHP
---

Here's an example: say we have a number (1) stored in a database table row, and we want to turn it into a human-readable day name, such as "Tuesday".

Here was my attempt at getting it done in PHP, after checking through the end of the manual for the obscure date functions:

``` php
// Method Signature (from http://www.php.net/manual/en/function.jddayofweek.php )
// mixed jddayofweek ( int $julianday [, int $mode = CAL_DOW_DAYNO ] )

// In use:
echo jddayofweek(1, CAL_DOW_LONG);  // Produces "Tue"
echo jddayofweek(1, CAL_DOW_SHORT); // Produces "Tuesday"
```

Ow. So what's going on?
<!--more-->

We have a "set" of calendar constants that would seem to apply:

* `CAL_DOW_DAYNO` (evaluates to `0`)
* `CAL_DOW_SHORT` (evaluates to `1`)
* `CAL_DOW_LONG` (evaluates to `2`)

Looking at the method signature, `jddayofweek` clearly uses `CAL_DOW_DAYNO`, which would imply that `CAL_DOW_LONG` and `CAL_DOW_SHORT` are also usable. Right? Nope.

It turns out `jddayofweek` doesn't use anything other than `CAL_DOW_DAYNO`. `1` returns the full day, and `2` returns an abbreviated day. `jddayofweek` doesn't use the rest of the `CAL_DOW_...` constants at all, just the plain numbers instead.

## Learnings

This is a good argument for symbols a la Ruby (or atoms as per Erlang, or similar concepts across a host of other languages). 

### Constants

With PHP (and C, Ruby, Perl and a bunch of other languages), constants are thin veils draped over values like integers and strings; you can see right through them if you get close enough. Several PHP constants could evaluate to the same value. Here's a sampler of constants that evaluate to `2`:

* `CAL_DOW_LONG`
* `CAL_EASTER_ALWAYS_GREGORIAN`
* `IMG_JPG`
* `MYSQLI_NUM`
* `X509_PURPOSE_SSL_SERVER`
* `EXTR_PREFIX_SAME`
* `E_WARNING`

Say we have this function:

``` php
const("STATUS_PEACE", 1);
const("STATUS_WAR", 2);
function should_launch_nukes($status) {
  return ($status == STATUS_WAR);
}
```

These are functionally equivalent:

``` php
// All indicate we should launch the nukes.
echo should_launch_nukes(STATUS_WAR);
echo should_launch_nukes(CAL_DOW_LONG);
echo should_launch_nukes(MYSQLI_NUM);
echo should_launch_nukes(2);
```

### Ruby Symbols

With Ruby, a symbol represents something else; it's a constant with a name the same as its value. A function that accepts `:jpg` is never going to accept a symbol like `:dow_long` and confuse it with `:jpg`, `:png`, etc. As far as the developer typing in the code cares, `:jpg` means neither `2` nor `"jpg"`; it just means `:jpg`.

``` ruby
def launch_nukes?(status)
  status == :war
end

launch_nukes?(:peace)    # => Nope.
launch_nukes?(:dow_long) # => Nope.
launch_nukes?(2)         # => Nope.
launch_nukes?(:war)      # => Fire ze missiles.
```


### Strings vs Symbols

So why not strings instead? `launch_nukes?("peace")` would be just as good, right?

With the bonus semantic benefits aside, symbols are just plain efficient. `:jpg` is `:jpg` is `:jpg`. Inspect a bunch of `:jpg` symbols, and you'll find they'll all be references to the one object. Try this out:

``` ruby
puts :jpg.object_id # => 454088
puts :jpg.object_id # => 454088
puts :jpg.object_id # => 454088
```

(This is, though, arguably a problem with Ruby. Python treats strings as immutable, and are thus the same as symbols in this regard. Ruby strings are mutable (eg. `str = "foo"; str << "bar"; puts str` outputs "foobar"), so we need a separate classification for the immutable variant. Consider, though, that PHP strings are mutable, *and* there is no separate symbol-alike type.)

[This article by Robert Sosinski](http://www.robertsosinski.com/2009/01/11/the-difference-between-ruby-symbols-and-strings/) goes into greater depth about the inner workings of symbols and how they differ from strings. (Cheers to Mr Sosinski for the idea for the Nukes example.)
