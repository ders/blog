+++
date = "2013-02-20T10:58:00+09:00"
title = "The RubyMonk Tutorial"

+++

I've been working my way through the [RubyMonk Tutorials](http://rubymonk.com/learning/books).
They are very well written and do an excellent job of guiding the user through progressively harder concepts.

Today I found a couple of careless coding errors in the [section on using Array#inject](http://rubymonk.com/learning/books/4/chapters/44-collections/lessons/98-iterate-filtrate-and-transform) (subscription required).  The first is an illustration of inject to do a simple sum of array elements.

```
[4, 8, 15, 16, 23, 42].inject(0) do |accumulator, iterated|
  accumulator += iterated
  accumulator
end
```

In fact, the assignment statement here is superfluous, as the inject
function assigns the return value of the block to the accumulator automatically.
It would suffice to do:

```
[4, 8, 15, 16, 23, 42].inject(0) do |accumulator, iterated|
  accumulator + iterated
end
```

The second error is in an illustration of the behavior of inject.
The function below is supposed to mimic the behavior of the one above.

```
def custom_inject(array, default = nil)
  accumulator = default || array[0]

  array.each do |element|
    accumulator = accumulator + element
  end

  accumulator
end

p custom_inject([4, 8, 15, 16, 23, 42], 0)
```

The `custom_inject` function, however, fails for the case where the default value is not given.  It fails by counting the first array element twice.

Native inject begins iterating from the second array element, as the first element has already been used as the starting value.
