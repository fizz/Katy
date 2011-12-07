Katy: CoffeeScript Combinators
===

Katy makes writing [fluent][fluent] CoffeeScript (and JavaScript!) easy by providing the `.K` and `.T` combinators for ordinary classes and objects.

The **tl;dr** is that Katy adds two methods, `.K` and `.T` to any classes you desire:

```CoffeeScript
KT = require('Katy').KT

KT.mixInto(String)

# K calls a function on the receiver and returns the receiver

'Hello'.K( (s) -> s + ' World' )
  # => returns 'Hello'
  
# T calls a function on the receiver and returns the result

'Hello'.T( (s) -> s + ' World' )
  # => returns 'Hello World'
```

You can also call any method by name:

```CoffeeScript
KT.mixInto(Array)

[1..10]
  .K('pop')
  .K('pop')
  .K('pop')
  .T('pop')
  # => returns 7
```

Snarf the source here, or install it with `npm install Katy`.

## How does that make my code more fluent?

You're familiar with [fluent interfaces][fluent]. They're great, but they rely on the author of the API making sure that each function returns its receiver. The `.K` method allows you to make any function or method "fluent" even if the original author has other ideas. The `.K` and `.T` methods also allow you to write your own methods and 'call' them just as if they were baked into the original object. For example, you can fake an `identifiers` filter for arrays of strings:

[fluent]: http://en.wikipedia.org/wiki/Fluent_interface

```CoffeeScript
require 'underscore'

identifiers = (arrOfSymbols) ->
  _.select arrOfSymbols, (str) ->
    /^[_a-zA-Z]\w*$/.test(str)
  
idents = someArray.T(identifiers)
```

This is cleaner than trying to mix ordinary functions with methods and adopting temporary variables when you want to work around what the function was written to return. In this example, having extended `Array.prototype` with `.K` and `.T` once, you need not extend it any more to add your own custom methods.

To recap:

1. You can make any function into something that can be called like a method, making your code read more naturally, and;
2. You can give any function or built-in method either "fluent" (return the receiver) or "pipeline" (return its value) semantics as you please.

## Monkey-patching is evil!

No problem. You don't need to mix it into a class, you can "wrap" an object without altering its prototype:

```CoffeeScript
KT('Hello')
  .K( (s) -> s + ' World' )
    # => returns 'Hello'

KT('Hello')
  .T( (s) -> s + ' World' )
    # => returns 'Hello World'
```

You can also 'chain' multiple invocations in a style borrowed from [Monads][m] and [Underscore][u]:

[m]: http://en.wikipedia.org/wiki/Monad_(functional_programming)
[u]: http://documentcloud.github.com/underscore/

```coffeescript
KT([1..10])
  .chain()
    .K('pop')
    .K('pop')
    .K('pop')
    .T('pop')
      .value()
  # => returns 7
```
  
## Is it any good?

[Yes][y].

## String Lambdas

As you know, Katy accepts any function as an argument to `.K` and `.T`, as in: 

```coffeescript
KT('Hello')
  .T( (s) -> s + ' World' )
    # => returns 'Hello World'
```

CoffeeScript makes writing functions pretty easy, but if you're using JavaScript, writing short functions is not quite as much fun:

```javascript
KT('Hello')
  .T( function (s) { return s + ' World'; } )
    // => returns 'Hello World'
```

The ceremonial trappings overwhelm the logic of what you're writing. That's doubly painful since you're already adding some extra indirection by using Katy. So, Katy lets you define one-liner functions using a highly abbreviated syntax invented by [Oliver Steele][osteele] called "String Lambdas." Here's the cheat sheet:

```javascript

// ->
//
// Like CoffeeScript, Katy supports using -> in a string. You don't need to parenthesize
// the arguments and no explicit "return" is needed.

KT('Hello').T( "s -> s + ' World'" ) // => returns 'Hello World'
  
// implicit parameters
//
// If you don't use '->', Katy will attempt to infer the parameters in your expression
// from left to right:

KT('Hello').T( "str + ' World'" ) // => returns 'Hello World'
  
// _
//
// If you use an underscore by itself, Katy will assume that's your only parameter.
// Yes, this conflicts with the Underscore library, but that's ok: String Lambdas
// are best for short, simple things. Note that in this example, Katy knows not
// to assume that 'world' is a parameter.

window.world = 'World'

KT('Hello').T( "_ + ' ' + world" ) // => returns 'Hello World'

// point-free
//
// For certain very simple expressions, Katy will assume parameters even if you don't supply them!

KT('Hello').T( "+ ' World'" )  // => returns 'Hello World'
KT('Hello').T( "+", ' World' ) // => returns 'Hello World'
  
// This works for simple message sending and property access as well:

KT('Hello').T( ".toUpperCase()" ) // => 'HELLO'

KT('Hello').T( ".length" ) // => 5
```

## String lambdas? You can't be serious!?

[Oh yes I can][sl].

[sl]: https://github.com/raganwald/homoiconic/blob/master/2008-11-28/you_cant_be_serious.md

[osteele]: http://osteele.com/

## Stuff and nonsense, this is a syntax issue, not a functional issue

[I agree][sans-titre], but that being said:

1. You can use Katy *now* instead of waiting to see if CoffeeScript adopts [a syntax for chaining methods][1495], and;
2. The `.K` and `.T` methods turn any function into something you can call like a method, which makes your code read more cleanly.

[sans-titre]: https://github.com/raganwald/homoiconic/blob/master/2011/11/sans-titre.md "Sans Titre"
[1495]: https://github.com/jashkenas/coffee-script/issues/1495 "Improve chaining syntax"

[y]: http://news.ycombinator.com/item?id=3067434

[um]: https://github.com/raganwald/Underscore-Matchers-for-Jasmine

## Cool! Does it work with jQuery?

Yes, but if you like jQuery and like Katy, you'll love [jQuery Combinators][jc].

[jc]: https://github.com/raganwald/JQuery-Combinators

## What's with the naming conventions?

`.T` is known in some CS circles as the [Thrush][t] or `T` combinator. Likewise, `.K` is known in combinatory logic circles as the "K Combinator" or [Kestrel][k]. To simplify the explanation radically, `T` and `K` are called combinators because they combine things to produce a result in different ways. Functional programmers call such things higher-order functions, but what makes combinators interesting is that combinators work by rearranging the order of things in an expression.

For example, `T` reverses the order of two things. Think about it: Instead of writing `identifiers(some_array)`, we use `T` to write `some_array.T(identifiers)`. That rearrangement is very handy for making our code conform to fluent style. Likewise, `K` leaves them in the same order but removes something. This ability to rearrange things is what makes them so useful for taking code that would normally have function calls sprinkled throughout it and rearranging it into a nice tree of method calls in fluent style.

Many other combinators exist, and they are all interesting with applications for functional and OO programmers. With combinators you can even get rid of parentheses in a programming language! If you aren't familiar with Combinatory Logic, I encourage you to follow the links to my posts about Kestrels and Thrushes, and better still do a little digging about Combinatory Logic in general. It's a rich, fascinating field of study that is so simple it's incredibly easy to pick up, and it leads naturally into functional and [concatenative][joy] languages.

[k]: http://github.com/raganwald/homoiconic/blob/master/2008-10-29/kestrel.markdown#readme
[t]: http://github.com/raganwald/homoiconic/blob/master/2008-10-30/thrush.markdown#readme
[joy]: http://github.com/raganwald/homoiconic/blob/master/2008-11-16/joy.md#readme