---
title: Functional-ish Ruby
---

In a [recent Apprentice Blog of the Week](http://blog.8thlight.com/alex-hill/2014/07/10/useful-clojure-macros-for-the-object-oriented-programmer.html), Alex Hill detailed one way that we can apply common Ruby patterns to our Clojure code. I’ve noticed a similar effect while making the opposite transition, too. Having spent a few months writing mostly Clojure and then transitioning back into writing mostly Ruby, it was interesting to see the way my experience with common patterns in Clojure influenced the way I approached writing Ruby.

Specifically, a couple of patterns I really enjoy using in Clojure are `with` and `when` macros. Usually, a macro starting with `with-` means that something is happening around the code you pass to it, and a macro starting with `when-` means that your code will be executed if a certain condition is met. For instance, we could write a macro called `with-timing` that times our code by setting a start time, evaluating the code, then logging the difference between the start and end times before returning our return value.

```clojure
(defmacro with-timing [body]
  `(let [start# (now)
         ret# ~body]
     (logger/log (- (now) start#))
     ret#))

(defn timed-operation [x]
  (with-timing
    (calculate-some-things x)))
```

We might also write a macro called `when-valid`, which takes a record that we created from some user input, and then only evaluates our code if the record is valid, otherwise using the generic handler for invalid records.

```clojure
(defmacro when-valid [record & body]
  `(if (valid? ~record)
     (render-invalid ~record)
     ~@body))

(defn response-for [thing]
  (when-valid thing
    (render-created thing)))

```

We can implement our timing macro similarly in Ruby, by simply writing a method that takes a block to be called and timed.

```ruby
def with_timing(&block)
  start_time = Time.now
  return_value = block.call
  log(Time.now - start_time)
  return_value
end

def timed_operation(x)
  with_timing do
    calculate_some_stuff(x)
  end
end
```

We can also reduce the duplication of a common Rails controller pattern by writing something similar to our `when-valid` macro in Ruby.

```ruby
def when_valid(record, &block)
  if record.valid?
    block.call(record)
  else
    flash[:error] = record.errors.messages
    render :new
  end
end

def create
  when_valid(Thing.create(thing_params)) do |thing|
    redirect_to thing
  end
end
```

Here’s another useful `when` method for handling HTTP responses in Ruby that [Myles Megyesi](http://www.8thlight.com/team/myles-megyesi) shared with me.

```ruby
def when_status(response, responders)
  if responder = responders[response[:status]]
    responder.call(response)
  else
    handle_generically(response)
  end
end

def get_all_the_things
  when_status get("/things"), {
    200 => lambda do |response|
      load_things(response[:body])
    end,
    404 => lambda do
      "Whoops"
    end
  }
end
```

After transitioning back to writing Ruby after Clojure, I found myself naturally thinking of ways to use blocks and lambdas, among other functional-ish idioms, much more than before writing Clojure—and usually with positive results. It’s interesting to see how learning new languages expands the way you write the languages you already know. Perhaps there are patterns from certain languages you know just waiting to be applied somewhere else.
