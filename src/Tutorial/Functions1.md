# Functions Part 1

Idris is a *functional* programming language. This means,
that functions are its main form of abstraction (unlike for
instance in an object oriented language like Java, where
*objects* and *classes* are the main form of abstraction). It also
means that we expect Idris to make it very easy for
us to compose and combine functions to create new
functions. In fact, in Idris functions are *first class*:
Functions can take other functions as arguments and
can return functions as their results.

We already learned about the basic shape of top level
function declarations in Idris in the [introduction](Intro.md),
so we will continue from what we learned there.

```idris
module Tutorial.Functions1
```

## Functions with more that one Argument

Let's implement a function, which checks if its three
`Integer` arguments form a
[Pythagorean triple](https://en.wikipedia.org/wiki/Pythagorean_triple).
We get to use a new operator for this: `==`, the equality
operator.

```idris
isTriple : Integer -> Integer -> Integer -> Bool
isTriple x y z = x * x + y * y == z * z
```

Let's give this a spin at the REPL before we talk a bit
about the types:

```repl
Tutorial.Functions1> isTriple 1 2 3
False
Tutorial.Functions1> isTriple 3 4 5
True
```

As can be seen from this example, the type of a function
of several arguments consists just of a sequence
of argument types chained by function arrows (`->`), which
is terminated by a return type (`Bool` in this case).

Now, unlike `Integer` or `Bits8`, `Bool` is not a primitive
data type built into the Idris language but just a custom
data type that you could have written yourself. We will
learn more about declaring new data types in another
part of this tutorial.

### Function Composition

Functions can be combined in several way, the most direct
probably being the dot operator:

```idris
square : Integer -> Integer
square n = n * n

times2 : Integer -> Integer
times2 n = 2 * n

squareTimes2 : Integer -> Integer
squareTimes2 = times2 . square
```

Give this a try at the REPL! Does it do what you'd expect?

We could have implemented `squareTimes2` without using
the dot operator as follows:

```idris
squareTimes2' : Integer -> Integer
squareTimes2' n = times2 (square n)
```

It is important to note, that functions chained by the dot
operator are invoked from right to left: `times2 . square`
is the same as `\n => times2 (square n)` and not
`\n => square (times2 n)`.

We can conveniently chain several functions using the
dot operator to write more complex functions:

```idris
dotChain : Integer -> String
dotChain = reverse . show . square . square . times2 . times2
```

This will first multiply the argument by four, then square
it twice before converting it to a string (`show`) and
reversing the resulting `String` (functions `show` and
`reverse` are part of the Idris prelude and as such are
available in every Idris program).

### Higher Order Functions

Functions can take other functions as arguments. This is
an incredibly powerful concept and we can go crazy with
this very easily. But for sanity's sake, we'll start
slowly:

```idris
isEven : Integer -> Bool
isEven n = mod n 2 == 0

testSquare : (Integer -> Bool) -> Integer -> Bool
testSquare fun n = fun (square n)
```

First `isEven` uses the `mod` function to check, whether 
an integer is divisible by two. But the interesting function
is `testSquare`. It takes two arguments: The first argument
is of type *function from `Integer` to `Bool`*, and the second
of type `Integer`. This second argument is squared before
being passed to the first argument. Again, give this a go
at the REPL:

```repl
Tutorial.Functions1> testSquare isEven 12
True
```

We can use higher order functions (functions taking other
functions as their arguments) to build powerful abstractions.
Consider for instance the following example:

```idris
twice : (Integer -> Integer) -> Integer -> Integer
twice f n = f (f n)
```

And at the REPL:

```repl
Tutorial.Functions1> twice square 2
16
Tutorial.Functions1> (twice . twice) square 2
65536
Tutorial.Functions1> (twice . twice . twice . twice) square 2
*** huge number ***
```

You might be surprised about this behavior, so we'll try
and break it down. The following two expressions are identical
in their behavior:

```idris
expr1 : Integer -> Integer
expr1 = (twice . twice . twice . twice) square

expr2 : Integer -> Integer
expr2 = twice (twice (twice (twice square)))
```

So, `square` raises its argument to the 2nd power,
`twice square` raises it to its 4th power,
`twice (twice square)` raises it to its 16th power,
and so on, until `twice (twice (twice (twice square)))`
raises it to its 65536th power resulting in an impressively
huge result.

### Currying

Once we start using higher order functions, the concept
of partial function application (also called *currying*
after mathematician and logician Haskell Curry) becomes
very important.

Load this file in a REPL session and try the following:

```repl
Tutorial.Functions1> :t testSquare isEven
testSquare isEven : Integer -> Bool
Tutorial.Functions1> :t isTriple 1
isTriple 1 : Integer -> Integer -> Bool
Tutorial.Functions1> :t isTriple 1 2
isTriple 1 2 : Integer -> Bool
```

Note, how in Idris we can only partially apply a function
with more than one argument and as a result get a new function
back. For instance, `isTriple 1` applies argument `1` to function
`isTriple` and as a result returns a new function of
type `Integer -> Integer -> Bool`. We can even
use the result of such a partially applied function in
a new top level definition:

```idris
partialExample : Integer -> Bool
partialExample = isTriple 3 4
```

And at the REPL:

```repl
Tutorial.Functions1> partialExample 5
True
```

We already used partial function application in our `twice`
examples above to get some impressive results with very
little code.

## Exercises

1. Reimplement functions `testSquare` and `twice` by using the dot
operator and dropping the second arguments (have a look at the
implementation of `squareTimes2` to get an idea where this should
lead you). This highly concise
way of writing function implementations is sometimes called
*point-free style* and is often the preferred way of writing
small utility functions.

2. Declare and implement function `isOdd` by combining functions `isEven`
from above and `not` (from the Idris prelude). Use point-free style.

3. Declare and implement function `isSquareOf`, which checks whether
its first `Integer` argument is the square of the second argument.

4. Declare and implement function `isSmall`, which checks whether
its `Integer` argument is less than or equal to 100. Use one of the 
comparison operators `<=` or `>=` in your implementation.

5. Declare and implement function `absIsSmall`, which checks whether
the absolute value of its `Integer` argument is less than or equal to 100.
Use functions `isSmall` and `abs` (from the Idris prelude) in your implementation,
which should be in point-free style.

6. In this slightly extended exercise we are going to implement
some utilities for working with `Integer` predicates (functions
from `Integer` to `Bool`). Implement the following higher order
functions (use boolean operators `&&`, `||`, and function `not` in
your implementations):

```idris
-- return true, if and only if both predicates hold
and : (Integer -> Bool) -> (Integer -> Bool) -> Integer -> Bool

-- return true, if and only if at least one predicate holds
or : (Integer -> Bool) -> (Integer -> Bool) -> Integer -> Bool

-- return true, if the predicate does not hold
negate : (Integer -> Bool) -> Integer -> Bool
```

After solving exercise 6, give it a go in the REPL. In the
example below, we use binary function `and` in infix notation
by wrapping it in backticks. This is just a syntactic convenience
to make certain function applications more readable:

```repl
Tutorial.Functions1> negate (isSmall `and` isOdd) 73
False
```

7. Idris allows us to define our own infix operators. In
fact, all infix operators we encountered so far (`==`, `&&`,
`||`, `.`, and so on) are custom operators defined in the
Idris prelude and not built into the language itself.
Even better, Idris supports *overloading* of function names,
that is, two functions or operators can have the same
name, but different types and implementations.
Idris will make use of the
types to distinguish between equally named operators and
functions.

This allows us, to reimplement functions `and`, `or`, and `negate`
from Exercise 6 by using the existing operator and function
names from boolean algebra:

```idris
-- return true, if and only if both predicates hold
(&&) : (Integer -> Bool) -> (Integer -> Bool) -> Integer -> Bool
x && y = and x y

-- return true, if and only if at least one predicate holds
(||) : (Integer -> Bool) -> (Integer -> Bool) -> Integer -> Bool

-- return true, if the predicate does not hold
not : (Integer -> Bool) -> Integer -> Bool
```

Operator names have to be wrapped in parentheses unless they
are being used in infix notation (see the `(&&)`) example above.

Implement the other two functions and test them at the REPL:

```repl
Tutorial.Functions1> not (isSmall && isOdd) 73
False
```

## Conclusion

What we learned in this part of the tutorial:

* A functions in Idris can take an arbitrary number of arguments,
separated by `->` in the function's type.

* Functions can be combined
sequentially using the dot operator, which leads to highly
concise code.

* Functions can be partially applied by passing them fewer
arguments than they expect. The result is a new function
expecting the remaining arguments. This technique is called
*currying*.

* Functions can be passed as arguments to other functions, which
allows us to easily combine small coding units to create
more complex behavior.

* Idris allows us to define our own infix operators. These
have to be written in parentheses unless they are being used
in infix notation.

* Idris supports name overloading: Functions can have the same
names but different implementations. Idris will decide, which function
to used based to the types involved.

Please note, that function and operator names in a module
must be unique. In order to define two functions with the same
name, they have to be declared in distinct modules. If Idris
is not able to decide, which of the two functions to use, we
can help name resolution by prefixing a function with
(a part of) its *namespace*:

```repl
Tutorial.Functions1> :t Prelude.not
Prelude.not : Bool -> Bool
Tutorial.Functions1> :t Functions1.not
Tutorial.Functions1.not : (Integer -> Bool) -> (Integer -> Bool) -> Integer -> Bool
```

<!-- vi: filetype=idris2
-->