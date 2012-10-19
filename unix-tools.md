# Learn Haskell as a Toolsmith

Bryan O'Sullivan

Facebook


# Before we begin...

I hope everyone has installed the Haskell Platform!

* [http://www.haskell.org/platform/](http://www.haskell.org/platform/)

If you have not, *please do now*

* Expect it to take 5-10 minutes


# What is the Haskell Platform?

A "batteries included" distribution

* Compiler, interpreter, other tools

* Core libraries for some common uses

* Documentation

Often just called "the HP"


# The main tools you'll use

The Platform ships with an optimising compiler

* GHC, the Glasgow Haskell Compiler

Also an interactive REPL

* GHCi


# Getting started

Open up a terminal window and start GHCi:

~~~~ {.haskell}
ghci
~~~~

It should print output like this:

~~~~ {.haskell}
GHCi, version 7.6.1: http://www.haskell.org/ghc/  :? for help
Loading package ghc-prim ... linking ... done.
Loading package integer-gmp ... linking ... done.
Loading package base ... linking ... done.
Prelude>
~~~~

That last line is its prompt.


# A familiar start

Basic arithmetic looks familiar:

~~~~ {.haskell}
Prelude> 2+3
5
~~~~

If you're pining for C, you can even write numbers in hexadecimal:

~~~~ {.haskell}
31337 * 0xdeadbeef
~~~~

Quick!

* What are the first and last digits of the result of the expression
  above?


# So long, ghci!

You can quit `ghci` as follows:

~~~~ {.haskell}
Prelude> :quit
~~~~

Or abbreviate:

~~~~ {.haskell}
Prelude> :q
~~~~

Or simply type control-D:

~~~~ {.haskell}
Prelude> ^D
~~~~


# Writing a tiny program

Open your text editor, and save the following to a file named
`HelloWorld.hs`:

~~~~ {.haskell}
main = putStrLn "hello world
~~~~

* `HelloWorld` - Haskell programmers use CamelCase naming

* File (module) names start with a capital letter

* `.hs` is the source file suffix


# Run it!

We can immediately run a program using the interpreter.

Try this from your terminal window:

~~~~
runghc HelloWorld
~~~~

As with most languages, the interpreter:

* Starts up quickly

* Runs slowly


# Compile it!

We prefer to run standalone native executables.

Here's how to build ours:

~~~~
ghc -O --make HelloWorld.hs
~~~~

* `ghc` runs the compiler

* `-O` means "enable optimisation"

* `--make` means "figure out dependencies, and build a binary"


# Run our compiled program!

After we run this...

~~~~
ghc -O --make HelloWorld.hs
~~~~

...There will be an executable named `HelloWorld` in the current
directory.

(`HelloWorld.exe` on Windows)

Run it:

~~~~
./HelloWorld
~~~~


# Quick pause

Has everyone been able to compile and run their `HelloWorld`
executable?


# Advertisement: Emacs

I prefer to develop my Haskell code using Emacs.

Haskell support:

~~~~ {.haskell}
git clone https://github.com/haskell/haskell-mode
~~~~

Simple setup for your `.emacs`:

~~~~ {.commonlisp}
;; Tell Emacs how to find the package
(add-to-list 'load-path "~/git/haskell-mode")

;; Load at startup
(load "haskell-site-file" nil t)

;; Make auto-indent do something halfway sane
(add-hook 'haskell-mode-hook 'turn-on-haskell-indent)
~~~~


# How I usually work

Open a source file in Emacs.

Hack, hack, hack.

Wonder if it's making sense.

Type `C-c C-l` to load it into `ghci`.

Interact with it, see if it made sense.


# Strings and lists

Back into `ghci`, please!

Here's a string:

~~~~ {.haskell}
"foo"
~~~~

And here's a list:

~~~~ {.haskell}
[1,2,3]
~~~~


# Appending, part 1

Please tell me what the result of this expression is:

~~~~ {.haskell}
"foo" ++ "bar"
~~~~


# Appending, part 2

And now this expression:

~~~~ {.haskell}
[1,2,3] ++ [4,5,6]
~~~~


# The implication?

A string is just a kind of list.

Try typing this into `ghci`:

~~~~ {.haskell}
['a','b','c']
~~~~

We have two kinds of quotes, like C:

* Single quotes: one character

* Double quotes: a string (list of zero or more characters)


# Simple logical operators

Try this in `ghci`:

~~~~ {.haskell}
"foo" == ['f','o','o']
~~~~



# Simple logical operators

Try this in `ghci`:

~~~~ {.haskell}
"foo" == ['f','o','o']
~~~~

It should evaluate to `True`.

Now this:

~~~~ {.haskell}
"foo" < "bar"
~~~~


# We just learned something else!

The Boolean values are named `True` and `False`.

* We call these *value constructors*, because they construct Boolean
  values.

Oh, and case is important!

* Try typing `true` into `ghci` instead.

* What error message do you get?


# Using functions

More fun in `ghci`.

What does this expression print?

~~~~ {.haskell}
last "bar"
~~~~


# How we use functions

*Unlike* C-family languages:

* We separate a function from its arguments using whitespace.

This is the first of *many* cases where:

* "Stuff that works in C"

does not translate into

* "Stuff that I should expect to work in Haskell"


# For example...

We've seen the equality operator:

* The familiar `==`

What about inequality?

* It's `/=`, sort of like $\neq$ in mathematics


# More complex expressions

Suppose we need to supply a function with a non-trivial argument

For example:

* The `reverse` function

Its argument:

* `"foo" ++ "bar"`

Try entering the two above into `ghci` *without* any parentheses.

What do you see?

Next, figure out where the parentheses should go to get a result of
`"raboof"`.


# Counting characters

Our first actual interactive program!

~~~~ {.haskell}
countChars xs = show (length xs)

main = interact countChars
~~~~


# Two definitions?

This is our first function definition:

~~~~ {.haskell}
countChars xs = show (length xs)
~~~~

On the left:

~~~~ {.haskell}
countChars xs = {- ... -}
~~~~

* A function name: `countChars`

* The name of an argument: `xs`

A function and its argument(s) are separated by spaces.




# Two definitions!

This is our first function definition:

~~~~ {.haskell}
countChars xs = show (length xs) ++ "\n"
~~~~

On the right:

~~~~ {.haskell}
{- ... -} = show (length xs)
~~~~

* `length` computes the length of a list

* `show` renders a Haskell value as a string


# Second definition

~~~~ {.haskell}
main = interact countChars
~~~~

The `interact` function:

* Reads input from stdin as a string

* Hands it to the function (for us, `countChars`)

* Writes the result to stdout


# Our first Unix utility!

Save this to `WC.hs`:

~~~~ {.haskell}
countChars xs = show (length xs) ++ "\n"

main = interact countChars
~~~~

Compile:

~~~~
ghc -O --make WC
~~~~

Run from your shell prompt:

~~~~
./WC < WC.hs
~~~~

What number does this print for you?


# What can lists actually contain?

Try this in `ghci`:

~~~~ {.haskell}
[True, False] ++ ['a', 'b']
~~~~

What do you get?


# What can lists actually contain?

Try this in `ghci`:

~~~~ {.haskell}
[True, False] ++ ['a', 'b']
~~~~

What do you get?


# What can lists actually contain?

Try this in `ghci`:

~~~~ {.haskell}
[True, False] ++ ['a', 'b']
~~~~

What do you get?

* A *type error*!


# Why a type error?

We know we can have lists that contain several different kinds of
data:

* Numbers

* Characters

* Booleans

Why not a *mix*?


# Step one: type signatures

~~~~ {.haskell}
a :: Char

a = 'x'
~~~~

The `::` notation introduces a *type signature*.

A type signature means:

* The *name* on the left of `::`...

* ...*has the type*...

* ...the *type* on the right


# Type constructors

The name `Char` is a *type constructor*.

* It constructs a type.

We only find type names on the RHS of type signatures.


# Constructors, constructors

Now something should be should be clear:

* Why we called a name like `True` a *value constructor*

We can construct both

* *values* (usually) and

* *types* (on the RHS of a type signature)


# Understanding constructors

Here's a reasonable question:

* "How do I know whether I am looking at a type constructor or a value
  constructor?"

The answer is easy:

* If it starts with a capital letter, it's a constructor

* If lowercase, it's a variable

* If it's somewhere on the right of `::`, it's a *type* constructor

* Otherwise, it's a *value* constructor


# For example...

~~~~ {.haskell}
-- value constructor    ::    type constructor

True                    ::    Bool

-- (Oh yes, "--" introduces a comment.)

-- (A comments runs until end of line.)
~~~~


# Lists

We write a list of characters as follows:

~~~~ {.haskell}
xs :: [Char]

xs = "foo"
~~~~

The square brackets around a type name mean "list of *that type*".


# Other type names

Most common built in types:

* `Bool`: Boolean (values: `True` or `False`)

* `Char`: Unicode character

* `Double`: double precision floating point

* `Int`: 32- or 64-bit integer (signed)

* `Integer`: arbitrary precision integer (signed)


# Static typing

When we write a Haskell expression or program:

* *Every component* has a type that is known *before* the program runs.

So Haskell's type system is *static*:

* We know everything important about types before a program ever
  executes.


# And yet...

For the first $n$ minutes of this tutorial, we wrote *no* type
signatures.

And our snippets and programs *still worked*.

* Why?

The compiler can *infer* the right types for us


# Example

Consider:

~~~~ {.haskell}
-- a :: Bool

a = True

b = not a
~~~~

The name `a` must have the type `Bool`

* Because the value constructor `True` constructs a value of type
  `Bool`

The function `not` accepts a `Bool` and returns a `Bool`

* So `b` must have the type `Bool` too!


# Wait, what?

We said:

* "The function `not` accepts a `Bool` and returns a `Bool`"

But we haven't talked about writing types for functions!

Here is the type signature for `not`:

~~~~ {.haskell}
--     param  ->  result
not :: Bool   ->  Bool
~~~~


# Lists once again

Now that we know how to read a type signature:

~~~~ {.haskell}
a :: [Char]

a = ['x', 'y']
~~~~

It should be clear why `a` cannot contain values that are not `Char`:

* Because our type signature says it can't!


# But ...

We know we can have lists like `[Bool]`, `[Int]`, and so on ...

* How do we generalize this idea?

Just as we do with a function


# Parameters for functions

Not sure what values a function should accept?

~~~~ {.haskell}
square x = x * x
~~~~

The variable `x` is a parameter that lets us supply different values.

Really obvious.


# Parameters for types

Don't know what type of value a list should store?

Instead of a specific type, we supply a *type parameter*.

~~~~ {.haskell}
length :: [a] -> Int
~~~~

This signature says:

* `length` will work for lists whose elements can be of any type


# Replacement

Suppose we want to get the length of a string.

~~~~ {.haskell}
length "foo"
~~~~

The value `"foo"` has the type `[Char]`.

To figure out the type of `length "foo"`:

* Simply replace `length`'s type parameter `a` with the type `Char`


# Inspecting a list

How to get the first element of a list:

~~~~ {.haskell}
head :: [a] -> a
~~~~

This signature says:

* Accept a list of some unknown type

* Return a value of *the same type* as inside the list


# Rendering things: can we do it?

Our knowledge of types so far:

* Concrete types: `Char`, `[Bool]`

* Parameterised types: `a` and `[a]`

But we don't know how to print values yet!


# Typeclasses

This declaration introduces a set of types:

~~~~ {.haskell}
class Show a where
    show :: a -> String
~~~~

We are saying:

* There exists a set of types named `Show`.

* Let's give a type from this set the name `a`.

* The type `a` defines a function named `show`.


# Instances

How is the `show` function defined for e.g. `Bool`?

~~~~ {.haskell}
instance Show Bool where
  show True  = "True"
  show False = "False"
~~~~


# But why?

The parametric type of `length` lets us compute the length of any list
without caring what type is contained inside.

~~~~ {.haskell}
length :: [a] -> Int
~~~~

The parametric type of `show` lets us render a value of any type
without knowing the precise type.

* (Provided the compiler can tell the type is an instance of `Show`.)


# A silly example

Look at this contrived function.

~~~~ {.haskell}
wat xs = "my " ++ show xs ++
         " is " ++ show (length xs) ++
         " elements long"
~~~~

Two things:

* It needs to render the length of `xs`

* It also needs to render `xs`


# A silly example

Look at this contrived function.

~~~~ {.haskell}
wat xs = "my " ++ show xs ++
         " is " ++ show (length xs) ++
         " elements long"
~~~~

Two things:

* We don't need to know the type of `xs`

* We *do* need to be sure it's in the `Show` class


# Typeclass constraints

Introducing the *constraint*:

~~~~ {.haskell}
wat :: Show a => [a] -> [Char]
~~~~

The `Show a =>` notation means:

* I don't know what type `a` is

* I need it to be an instance of `Show`


# And one more thing!

This is our first multiline function:

~~~~ {.haskell}
wat xs = "my " ++ show xs ++
         " is " ++ show (length xs) ++
         " elements long"
~~~~

Yes, Haskell is sensitive to white space!

After a declaration begins:

* Every line that is more indented than the first *continues* the
  current declaration


# Counting words

Here's our original word count program:

~~~~ {.haskell}
countChars xs = show (length xs)

main = interact countChars
~~~~

Modify it to count words instead.

Use the `words` function. To find out about it:

* Use the `:type` directive in `ghci`

~~~~
:type words
~~~~


# Counting lines

If you wanted, you could easily modify your program to count lines.

~~~~ {.haskell}
lines :: String -> [String]
~~~~


# Sugar

We know two different syntaxes for lists.

Brackets:

~~~~ {.haskell}
[True, False]
~~~~

Strings:

~~~~ {.haskell}
"wibble"
~~~~

These are both syntactic sugar for a simpler form.


# Constructing lists

We always write the empty list as simply

~~~~ {.haskell}
[] :: [a]
~~~~

This is one of the list type's two constructors.

The other is an operator named "`:`".

~~~~ {.haskell}
(:) :: a -> [a] -> [a]
~~~~


# What's in a signature? I

~~~~ {.haskell}
(:) :: a -> [a] -> [a]
~~~~

This is the first type signature we have seen for a function of more
than one argument.

* The last item in the chain of `->` is the return type.

* The rest are parameter types.


# What's in a signature? II

~~~~ {.haskell}
(:) :: a -> [a] -> [a]
~~~~

This is also the first type signature we have seen for an *operator*.

* We simply wrap the operator name in parentheses to write a
  signature.

Another example:

~~~~ {.haskell}
class Eq a where
    (==) :: a -> a -> Bool
~~~~


# Sugar

You write:

~~~~ {.haskell}
[3]
~~~~

It means:

~~~~ {.haskell}
3 : []
~~~~


# Sugar

You write:

~~~~ {.haskell}
[2,3]
~~~~

It means:

~~~~ {.haskell}
2 : (3 : [])
~~~~


# Sugar

You write:

~~~~ {.haskell}
[1,2,3]
~~~~

It means:

~~~~ {.haskell}
1 : (2 : (3 : []))
~~~~


# Fixity

Infix operators have notions of *fixity* and *precedence*

* Where the magic parentheses get put when the real ones are missing

We're already familiar with this:

~~~~ {.haskell}
2 + 3 * 5 + 6
~~~~

With explicit parens:

~~~~ {.haskell}
(2 + (3 * 5)) + 6
~~~~


# Fixity and (:)

Lucky for us, `(:)` associates to the right.

So we can simplify this:

~~~~ {.haskell}
1 : (2 : (3 : []))
~~~~

To this:

~~~~ {.haskell}
1 : 2 : 3 : []
~~~~


# Where do values come from?

Suppose we construct a value like this:

~~~~ {.haskell}
1 : []
~~~~

The Haskell runtime has to manage this data for us.

It must remember that:

* We constructed an empty list

* We called the `(:)` constructor with 1 as its first parameter, and
  the empty list as its second


# What about us?

If the Haskell runtime has to manage this data for us, wouldn't it be
useful if we could inspect it?

We can---using *pattern matching*.

~~~~ {.haskell}
head (x:xs) = x
~~~~


# Pattern matching I

~~~~ {.haskell}
head (x:xs) = x
~~~~

The `(x:xs)` notation means:

* See if the value we are looking at was constructed using the `(:)`
  constructor.

* If it was, assign `x` to `(:)`'s first parameter, and `xs` to its
  second.


# Pattern matching II

~~~~ {.haskell}
myLength []     = 0
myLength (x:xs) = 1 + myLength xs
~~~~

Since a list has two constructors, it follows that:

* A function definition may need multiple clauses to match them all.


# How does pattern matching work?

Suppose we want to evaluate `myLength [1,2]`.

Pattern matching goes top to bottom.

~~~~ {.haskell}
myLength []     = 0
~~~~

Our value here is `1:(2:[])`.

The first constructor is `(:)`, not `[]`, so this pattern does not
match.



# How does pattern matching work?

Suppose we want to evaluate `myLength [1,2]`.

Our second pattern:

~~~~ {.haskell}
myLength (x:xs) = 1 + myLength xs
~~~~

Our value is still `1:(2:[])`.

This pattern matches.

We bind `x` to `1`, `xs` to `2:[]`, and evaluate the RHS:

~~~~ {.haskell}
1 + myLength (2:[])
~~~~


# Next step

Now we evaluate `myLength (2:[])`.

~~~~ {.haskell}
myLength []     = 0
~~~~

Still no match.

~~~~ {.haskell}
myLength (x:xs) = 1 + myLength xs
~~~~

Another match!

We bind `x` to `2`, `xs` to `[]`, and evaluate the RHS:

~~~~ {.haskell}
1 + myLength []
~~~~


# Where do we stand?

Two steps into evaluation, we now have this:

~~~~ {.haskell}
1 + (1 + myLength [])
~~~~

On the last application of `myLength`, we get:

~~~~ {.haskell}
1 + (1 + 0)
~~~~

~~~~ {.haskell}
1 + 1
~~~~

~~~~ {.haskell}
2
~~~~


# All right!

Here's that `myLength` function:

~~~~ {.haskell}
myLength []     = 0
myLength (x:xs) = 1 + myLength xs
~~~~

Modify it to compute the `sum` of a list of numbers.

~~~~ {.haskell}
mySum :: [Int] -> Int
~~~~


# Conditional execution

This syntax is called a guard:

~~~~ {.haskell}
isEven x
  | x `div` 2 == 0 = True
  | otherwise      = False
~~~~

Guards are evaluated top to bottom, if a pattern match succeeds.

The first guard to succeed has its RHS used as the result.


# uniq

The Unix `uniq` command gets drops consecutive repeated lines of
input.

~~~~
a
a
a
x
x
c
~~~~

~~~~
a
x
c
~~~~

Write a program that has the same behaviour.


# uniq

The Unix `uniq` command gets drops consecutive repeated lines of
input.

Write a program that has the same behaviour.

Useful functions:

~~~~ {.haskell}
lines :: String -> [String]

unlines :: [String] -> String
~~~~


# How I started

I knew I could use `lines` and `unlines` to convert into the form that
`interact` requires.

~~~~ {.haskell}
main = interact uniq

uniq xs = unlines (start (lines xs))
~~~~


# A "getting started function"

If there is no input, we do nothing.

~~~~ {.haskell}
start [] = []

start (x:xs) = go x xs
~~~~~~~~

Otherwise, get the first line of input, and call `go` on it.


# The common case

Go has a parameter, let's call it "the current line":

* The last non-repeated line we saw.

If we reach the end of input, yield the current line.

~~~~ {.haskell}
go x (y:ys)
   | x == y    = go x ys
   | otherwise = x : go y ys

go x _ = [x]
~~~~

If a new line is the same as the current line, do nothing.

If it differs, return the old current line and pass a new current
line to `go`.
