---
title: "Clojure Macros"
---

The project I’ve been pairing on lately uses quite a few macros. Sometimes I
understand what these macros do, and sometimes I don’t. I decided that I
basically just need to delve deeper into understanding macros in Clojure so
that I don’t run with any assumptions when I’m trying to understand a macro.
This post is my attempt to flesh out a little bit more about macros to myself.

### Macros vs. Functions

The key difference between macros and function in Clojure is that macros
execute at compile time, while functions execute at runtime. Macros produce
code at compile time. Functions produce data at runtime. Because Clojure is a
lisp, in which code can be manipulated like data, you can use a macro to
manipulate code at compile time, into new functions that get called at runtime.

### Quote and Unquote

How do we manipulate code in Clojure? Well, given that a function call is
essentially a list, eg. `(+ 1 2 3)`, we can manipulate that code by treating it
as data after applying a quote to it. We can do this through the explicit
`(quote (+ 1 2 3))`, or through using shorthand ’ (that’s a single apostrophe if
you can’t see it). A quoted function stays unevaluated and can be manipulated
like data.

When we need to evaluate certain things within the quoted code, we use the \`
(that’s a backtick) in place of quote. The backtick, known as a syntax quote,
allows us to evaluate things within the quoted expression by applying the ~ to
any expressions in the expression. For example, ``(+ 1 2 ~(+ 1 2))` would
evaluate to `’(+ 1 2 3)`, which can then be manipulated as data.

### Creating a Macro

We define a macro using defmacro. In its simplest form, a macro takes in an
expression, changes it, and returns the new expression. We could define a
simple macro called “fix” that takes some out of order code like `(1 + 1)` and
rearranges it to be `(+ 1 1)`.

### Macroexpand

I look at macroexpand as a function (a function, not a macro) that returns the
returned code from a macro. Macroexpand recursively evaluates any macros in the
provided expression it’s left with a pure function call. Because macroexpand is
a function and not a macro, the code passed as an argument needs to be quoted.

### Examples

Macros are used in some of the most common Clojure expressions. Some common
macros that I didn’t realize were just macros include defn, cond, and unless.
I’ve created some macros of my own to illustrate the core macro concepts that
I’ve described here. I think the key thing to keep in mind when trying to
understand macros is that a macros rearrange code and produce code at compile
time. Here’s my attempt at rearranging some code:

```clojure
(defmacro maybe...? [body]
  `(if ~(even? (rand-int 100))
    ~body
    "nope"))

(prn (macroexpand '(maybe...? (+ 2 2))))
(prn (maybe...? (+ 2 2 )))

(defmacro define-a-function [name args body]
  `(def ~name (fn ~args ~body)))

(prn (macroexpand '(define-a-function add [num1 num2] (+ num1 num2))))
(define-a-function add [num1 num2] (+ num1 num2))
(prn (add 1 1))
```
