---
layout: post
title:      "Sorting Object Properties In JavaScript"
date:       2018-07-26 12:27:00 -0400
permalink:  sorting_object_properties_in_javascript
---

The ordering of object properties in JavaScript is a subject with lots of history. Prior to ES6, properties of a JS object could not be expected to be returned in the same order as they were added when returned by functions like `Object.keys()` or `Object.getOwnPropertyNames()`. With the introduction of ES6, however, property key order was introduced. The properties are not necessarily returned in the order added, however. First, integer keys are returned, then strings in sorted order, then symbols.

So, basically, you still can't rely on your object properties being sorted when you call `Object.keys()`. So how would you return a sorted array of Object keys when needed?

One way is to use the `sort()` method on the array of property names returned by `Object.keys()`. The `sort()` method sorts the elements of the array in place and returns the array. `sort()` takes a callback function which is used to compare and sort the elements.

The callback function is the part that always confused me, but it's actually pretty simple. The return value of your callback when called for two elements (e.g. `a` and `b`) determines the sort order of your resulting array. Here are the rules:

If `callback(a, b)` returns:
- **Less than 0**, then `a` comes first in sort order
- **Exactly 0**, then `a` and `b` are equal (their relative positions will be unchanged in the sorted array)
- **Greater than 0**, then `b` comes first in the sort order

So to implement a sort of an object's keys by, e.g. the value of each key, all you have to do is the following:

```
let list = {'alligators': 13, 'zebras': 4, 'lions': 2, 'turtles': 12 }

Object.keys(list).sort((a, b) => {
  return list[a] - list[b];
});

//=> ["lions", "zebras", "turtles", "alligators"]
```

This returns the list of animals sorted by how many of them we have, in ascending order. Here we are saying to compare the values of the keys to each other. If `a` is less than `b`, the subtraction will result in a value less than 0, so then `a` will come first in sort order, according to the rules above. So this sorts our keys by value in ascending order, with the smaller values first. If we returned `list[b] - list[a]` instead, we would sort in descending order.

Now, what if we have some new animals but we have the same number of each new animal? Let's say in this case we would like to have those animals show up in alphabetical order in our sorted list. In this case we will need to customize our callback function further. If we just add our additional animals as follows, we'll see that `sharks` is coming before `monkeys`. Ideally, `monkeys` would come first as it starts with `m` and sharks starts with `s`.

```
let list = {'alligators': 13, 'zebras': 4, 'lions': 2, 'turtles': 12, 'sharks': 5, 'monkeys': 5 }

Object.keys(list).sort((a, b) => {
  return list[a] - list[b];
});

//=> ["lions", "zebras", "sharks", "monkeys", "turtles", "alligators"]
```

All that we have to do is add some logic to our callback to deal with keys having equal values.

```
Object.keys(list).sort((a, b) => {

  // if 2 keys have the same value
  if (list[a] - list[b] === 0) {

    // if a comes first alphabetically, return
    // -1 since it should come first in our sorted list
    if (a < b) {  
      return -1;
    } else {
      return 1;
    }

  // otherwise, sort by the value
  } else {
    return list[a] - list[b];
  }
});

//=> ["lions", "zebras", "monkeys", "sharks", "turtles", "alligators"]
```

Now we are sorting first by the key's value, then by the key's name!

It is important to note that the `sort()` method is not necessarily [stable](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability), so be careful when you have arrays that contain identical elements, though this shouldn't happen when sorting Object properties.

##### References:
- [MDN Array.prototype.sort()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
