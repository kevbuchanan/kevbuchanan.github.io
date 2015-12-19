---
title: Functional-ish Ruby, Part 2&#58; Functional Objects
---

In a [previous blog post](http://blog.8thlight.com/kevin-buchanan/2014/08/20/functional-ish-ruby.html)
I discussed a few ways to think about writing Ruby methods from a functional perspective. But in Ruby,
or any other object oriented language, we don't just write functions, we write objects. Does that mean
we either have to live without the principles we like from functional programming, or fundamentally
change the way we write Ruby or OO code? I'd argue, neither. Let's not conflate paradigms that we
have grown to like or dislike in OO and functional programming with objects themselves, or the lack thereof.

What are the benefits to programming functionally? There are many, and this blog isn't about
defining them, or validating what specific concepts actually define functional programming.
This blog assumes that there are certain ideas attached to the grander idea of
"functional programming," and that you see some value in them.

So, given that things like referential transparency, pure functions, immutable state,
currying, closures, higher order functions, and monads are things that we like,
what does that mean for those who predominately work in object oriented languages? Well, we can
still write Ruby, or any other object oriented language perfectly
fine while adhering to our favorite functional principles. Here are some examples:

## Immutable State and Avoiding Side Effects

Take the following object:

```ruby
class MutableObject
  attr_accessor :mutable_attrs

  def initialize(attrs = {})
    @mutable_attrs = attrs
  end

  def update(key, value)
    mutable_attrs[key] = value
  end
end

object = MutableObject.new
object.update(:some_key, :new_value)
# or
object.mutable_attrs[:some_key] = :new_value
```

In this class, not only does the ```update``` method change the state
of the object, but we've exposed our ```attrs``` hash through a public
```attr_accessor```, leaving it easily exposed for
mutation in place. What's the problem? Well, maybe nothing if the
scope of this object is localized enough so that everything using the object
knows that it could change. But once the mutations leave a defined
boundary, we've introduced more complexity to the codebase.

We don't need truly immutable data structures to fix this problem, we just
need to be mindful of when we actually should mutate state. If we decide that
mutating state causes unnecessary complexity, we could refactor
this class to look more like this:

```ruby
class MyObject
  def initialize(attrs = {})
    @attrs = attrs
  end

  def update(key, value)
    MyObject.new(attrs.merge(key, value))
  end

  private

  attr_reader :attrs
end

object = MyObject.new
new_object = object.update(:some_key, :new_value)
```

Now when we update an object we return a new instance. That way we know
that once we have an object, it won't change.

The nice thing about a functional language like Clojure is that we get
this for free.

```clojure
(def my-map {:number 1})

(def my-new-map (assoc my-map :number 2))

(:number my-new-map) ; => 2
(:number my-map) ; => 1
```

But, as explained before, sometimes mutation is ok, even better, if it's localized.
And even Clojure admits this by allowing a data structure to explicity be declared
as transient:

```clojure
(defn vec-range [n]
  (loop [i 0 v (transient [])]
    (if (< i n)
      (recur (inc i) (conj! v i))
      (persistent! v))))
```

State and mutability themselves aren't the problem. The problem is mutating across boundaries. And
we can address that by being aware of when we are mutating state, or hiding our objects' state
behind method calls rather than exposing public accessors.

### Referential Transparency in Enumerables

One pattern to be aware of with Ruby's enumberables is that the ```each``` method on its own
only returns an enumerable for the original collection. So, it can only have an
effect by changing state somewhere else and cannot be referentially transparent.

```ruby
class ThingFilter
  def initialize(things)
    @things = things
  end

  def call
    @things.each do |thing|
      thing.change!
    end

    @things
  end
end
```

Consider using ```map``` or ```reduce```, and if you can't, it's a sign that the operation
is mutating state, and is not referentially transparent or a pure function.

```ruby
def call
  @things.map do |thing|
    slightly_transform(thing)
  end.map do |thing|
    change_some_more(thing)
  end
end
```

```ruby
def call
  @things.reduce({}) do |name_hash, thing|
    name_hash[thing.id] = thing.name
    name_hash
  end
end
```

## Currying, Closure, and Function Instantiation

The important thing to remember about objects is that they are just functions that have
been instantiated with local bindings. ```MyObject.new(5)``` is just a function that
returns a new function to be called later with a local binding to 5.
To start, consider a class method that takes three arguments, and does
something immediately with the three arguments. It's clearly just a function
attached to a class namespace.

```ruby
class Foo
  def self.a_function(a, b, c)
    DoSomething.call(a + b + c)
  end
end

Foo.a_function(10, 11, 12)
```

But, what if we just want to bind two arguments and return a new function to be
called later? Well, then we instantiate a new object with two of our arguments,
which returns a new function to be called with one argument later on.

```ruby
class Foo
  def initialize(a, b)
    @a = a
    @b = b
  end

  def a_function(c)
    DoSomething.call(@a + @b + c)
  end
end

foo_fn = Foo.new(10, 11)
foo_fn.a_function(12)
```

In functional languages we do this through just returning a function, from a function.
Or, in Clojure, if we want to just return a function that takes one argument, we can use
the ```partial``` function.

```clojure
(defn foo [a b]
  (fn [c]
    (do-the-thing (+ a b c))))

((foo 1 2) 3)

(defn foo [a b c]
  (do-the-thing (+ a b c)))

((partial foo 1 2) 3)
```

To underscore the fact that functions are just objects with one method, Clojure's
functions are implemented in Java as an interface with one function, ```invoke```.

## Higher Order Functions

If objects are just functions, we can pass them around and return new functions.

### Composition

```ruby
class OperationManager
  def initialize(thing_doer)
    @thing_doer = thing_doer
  end

  def get_presenter(thing)
    result = @thing_doer.call(thing)
    ThingResultShower.new(result)
  end
end

operation = OperationManager.new(ThingDoer.new)
presenter = operation.get_presenter(my_thing)
presenter.show_result($stdout)
```

Here, we can look at the class ```OperationManager``` as a function that takes a ```thing_doer``` function,
then returns a new function that takes a ```thing``` argument. When ```get_presenter``` is called
with a ```thing```, it returns a function that can be called with an output stream to finally show
the result. In Erlang, this pattern might look like this:

```erlang
manage(Doer) ->
    fun(Thing) ->
        Result = Doer(Thing),
        fun(Out) -> write_out(Out, Result) end
    end.

do_something(MyThing) ->
    ActionFn = manage(fun my_doer_fn/1),
    PresenterFn = ActionFn(MyThing),
    PresenterFn("output.txt").
```

What's the benefit of this over one function that does everything at once? One, we've
made the object more composable by allowing how the thing gets done to be passed to it.
Two, perhaps the three arguments &mdash; the ```thing_doer```, the ```thing```, and the output
stream &mdash; are known at different places in our program, and are responsiblities of
different objects. We've now hidden the details of the two other arguments that
those objects don't care about.

### Monads

Another fun application of higher order functions is monads. A monad is basically
just a function that allows us to perform a connected series of operations
on an object. Consider the common "maybe" monad, which connects a series of
operations on a value that may or may not be present.

```ruby
class Maybe
  attr_reader :value

  def initialize(value)
    @value = value
  end

  def try
    if value.nil?
      Maybe.new(value)
    else
      result = yield value
      Maybe.new(result)
    end
  end
end

maybe = Maybe.new(:foo)
value = maybe.try { |x| x.to_s }.try { |x| nil }.try { |x| x + 1 }.value
```

The value here would be nil, but notice that there would not be an exception when trying to perform
```x + 1``` when x is nil.

In languages like Haskell or [Rust](http://blog.8thlight.com/felipe-sere/2015/03/02/porting-selecta.html),
we get something like this for free. In Clojure, a rudimentary maybe monad might look like this:

```clojure
(defn maybe [x]
  (fn
    ([op] (if (nil? x) (maybe x) (maybe (op x))))
    ([] x)))

(((((maybe :foo) str) (fn [x] nil)) (fn [x] (+ 1 x))))
```

Lately, I've been experimenting with a result monad, which is useful for eliminating nil checks
or abstracting event statuses or reasons for failure. It's not much different than the maybe
monad, the only difference is that we check for matching statuses, rather than nil.

```ruby
class Result
  attr_reader :result

  def self.status(status, result)
    new(status, result)
  end

  def initialize(status, result)
    @status = status
    @result = result
  end

  def on_status(status)
    if @status == status
      value = yield @result
      Result.status(@status, value)
    else
      Result.status(@status, @result)
    end
  end
end

result = Result.status(:not_found, nil)

title = result.on_status(:success) do |post|
  post.title
end.on_status(:not_found) do
  "Title not found"
end.result
```

## Conclusion

So, what does this mean? To me, it means that if we want to write functional code in an OO language
we just need to ask ourselves, "Could I easily model this object as a function?" If the answer is no,
then that's probably a sign that it's changing too much state, or is not composable, or is not
meeting some definition of the word functional. The ideas that we like about functional
programming can take many different forms, and these ideas don't have to be a prescriptive ideology.
There are obviously situations where enforcing immutable state or using a monad
makes our code more complex. But, if you see value in using immutable state, or higher
order functions, or a monad, then use it. Take the pieces you like when they fit, and find the
alternative best option when they don't.
