---
title: "Be Empirical"
---

>_One must learn by doing the thing; for though you think you know it you have no certainty, until you try._
>_&mdash; Sophocles_

It was well past the time we thought we'd leave that night. We'd been working for a whole week already, putting in 12 hour days, and were waiting with anticipation to see if the API we built would work. A couple of our mentors had gathered around to see the results. We went to write the cURL request with our API key and&hellip; wait&hellip; how do you add an authentication header to a cURL request again? Like this&hellip;? Or, like this&hellip;? Does it need to be capitalized? So should I put quotation marks around it? I think you need to put that in front actually. No&hellip;? I'm not sure either. Another mentor, an experienced developer, walked in. We asked him. "I don't remember. Just be empirical," he said bluntly. It was such a simple directive - just try something - but the beauty was in its simplicity and it was a profound piece of advice. Why are we looking around asking each other how to do it, when we can just try it out and let the computer tell us if we did something wrong?

Often, I find myself in that same position again. Asking myself if this is the right piece of code to write, or wondering what would happen if I try something another way. The bad part isn't the wondering, it's the wondering as if, "Oh well, guess I'll never be able to figure that out. Better Google for a Stack Overflow answer. I wish I had some way to see whether I can do that." Then, sometimes, in my more enlightened state, I think, "Well, let's just try it and find out." And at that point I get empirical data to back up or reject any claims that I had made about the code. In being empirical by not being afraid to break things, I can know for a fact if something works or doesn't work, and by paying attention to the right error message, I probably know why it doesn't work.

### If It's Not Broken, Break It

How do you typically start out with a new language or tool? Do you read about it? Or just start writing? In my experience, I learn faster by getting straight to work on a kata or toy program. This usually involves breaking something at every step and reading a lot of error messages. The notion of having a "breakable toy" when learning a new language, or just learning to program in general, has been a common theme in my learning process. At 8th Light, we all have a standard breakable toy that we use throughout our apprenticeship: Tic Tac Toe. This is a great way to learn a new language, but the term "breakable toy" downplays the importance of actually *breaking* the toy. This program isn't just ok to break, it is encouraged that you do. It *must* be broken, and you *must* continuously fix it. Without breaking your toy, you only find out half of the story - what works. You don't find out what doesn't work and why.

You may be familiar with this concept if you've read any of Zed Shaw's [Learn Code the Hard Way series](http://learncodethehardway.org/). He commonly gives you ways to break the code you just learned, and see what happens. Why? Well, if all you do is follow the directions, you may be able to remember how to do something similar in a future project, but chances are you'll start off with something that's broken, or a failing test, and you'll need to fix it. You can trust that he told you the one and only way to do something, or you can break it and find out yourself. You shouldn't hesitate to take code that you normally execute like this:

{% highlight ruby %}
numbers.each_with_index do |x, i|
  collection << [x, i]
end

return collection
{% endhighlight %}

and try something different, that you think might work, like this:

{% highlight ruby %}
numbers.map_with_index do |x, i|
  [x, i]
end
{% endhighlight %}

Because you'll immediately find out if it works or not (I'm close, but I'll need to try again on that last one). This is being empirical at its finest.

### Read the Error

I feel bad for error messages. They're so undeservedly vilified. Everyone makes them out to be the bad guy, the one they never want to see. When in reality, they're a great feedback loop, and are just trying to help you out - give you some information about what you did wrong. Sometimes, in my rush to make a test pass, I overlook this helpful error message that's trying to tell me something. I push him aside as if I know what's going on and continue down the path that I thought was right. Then, I've cast aside my empiricism. I've started to ignore the best feedback loop I have. I then spend 15 minutes trying to fix a problem that didn't even exist in the first place.

Part of being empirical involves not just seeing what doesn't work, but paying attention to the why when it presents itself. When I work with novice programmers, the biggest weakness I usually see is a lack of care in reading error messages. Your code is trying to tell you where you went wrong, even telling you what line and method invocation. For instance, if I think about what this error means:

{% highlight bash %}
src/player.c:5:9: error: conflicting types for 'create_player'
Player* create_player(char* piece, MoveHandler move_fn) {
        ^
include/src/player.h:13:9: note: previous declaration is here
Player* create_player(char piece, MoveHandler move_fn);
        ^
src/player.c:7:17: warning: incompatible pointer to integer conversion
                   assigning to 'char' from 'char *';
                   dereference with * [-Wint-conversion]
  player->piece = piece;
                ^ ~~~~~
                  *
1 warning and 1 error generated.
{% endhighlight %}

I should now know a little bit more about the language I'm using, namely that pointers to chars and chars are not the same thing, that you dereference with *, and that the compiler checks the types declared in my header file with definitions in my source.

If you listen to the errors you encounter, rather than assuming you know where you went wrong, or just disregarding it altogether, you'll take advantage of that feedback loop and figure out the what and why faster.

### Use Your Feedback Loops

In their book, [Apprenticeship Patterns](http://chimera.labs.oreilly.com/books/1234000001813/index.html), Dave Hoover and Adewale Oshineye describe objective, consistent, feedback loops as an integral part of the learning process.

>_Create mechanisms for regularly gathering more or less objective external data about your performance. By soliciting feedback early, often, and effectively, you increase the probability that you will at least be conscious of your incompetence._

Too often we forget the instant feedback loops that we have at our fingertips. We have the luxury of working with tools that are not bound by the constraints of other mediums. If I was an architect, I couldn't simply say, "This house looks nice, but I wonder what would happen if I changed the roof to look this way," and then do it. At least not without significant costs in both time and money. But as a software developer I can think of an idea to try, and with absolutely no research, know immediately if it works or why it doesn't work.

If my feedback loop is my test that I had written, I know what that change did wrong, versus what was expected. If my feedback loop was an error before my code even reached the test, I probably know even more about how the language works if I pay attention to the feedback the compiler or interpreter is giving me.

Feedback loops are the key to learning. The faster our feedback loops, the more information we process about what we're doing right and what we're doing wrong, and the faster we learn. The more times our feedback loops tell us we're wrong, the more improvements we make, the better we get. Learning by empiricism, by doing while paying attention to feedback loops, is [significantly more effective](http://www.simplypsychology.org/learning-kolb.html) than learning by simply reading, or watching, or listening. You can't get any better "objective external data" than actually doing the thing you want to do, and observing the result. And with code, you become immediately aware of your incompetence, or your correctness.

### Learn Through Empiricism

>_What is it we want to get out of code? The most important thing is learning. The way I learn is to have a thought, then test it out to see if it is a good thought. Code is the best way I know of to do this. Code isn't swayed by the power and logic of rhetoric. Code isn't impressed by college degrees or large salaries. Code just sits there, happily doing exactly what you told it to do. If that isn't what you thought you told it to do, that's your problem._
>_&mdash; Kent Beck, Extreme Programming Explained_

When learning, let's not forget about how much our code and tools can teach us. It's nice to have references like Stack Overflow when stuck, and mentors to help us critique our design, but the quickest feedback loop you have, and the best way to learn, empirically, is through your code. You'll learn more, faster, if you remove that voice inside your head that doesn't want to be wrong, that panics or gets frustrated when you read an error, and that doesn't want to break things. The more often you're wrong, and read errors, and break things, the better you get, and you'll be more confident in what you know, because you saw it happen.

