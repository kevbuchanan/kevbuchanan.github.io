---
title: Mistaking Encapsulation for Abstraction
---

"Abstract" is a commonly used word in software. I hear it a few times every day.
*"Can we make an abstraction around this?" "Let's move this method out to abstract it away."*
But, what are we really doing when we create an "abstraction" or when we "abstract"?
Are we actually doing what we think we're doing? What do these terms actually
mean, and are we using them correctly?

I'd argue we, including myself, use them too often, and often incorrectly. To create an abstraction, as explained
by [Rich Hickey](http://www.infoq.com/presentations/Simple-Made-Easy), means to draw away,
pull apart, or, to use his signature term, decomplect. To truly abstract things away in our
programs is an act that disentangles the complexity of our software, and further separates the
*who* from the *what*, *when*, *where*, and *why*. Abstraction should not be confused with code
organization and a mere hiding of complexity. Abstraction's goal is simplicity, not ease of use.

Hindering that goal is the fact that abstraction is commonly confused with encapsulation. If we come upon a complicated
class, we frequently just move some logic out, hide it in another class, and call it "abstracted."
However, this usually doesn't result in us taking any steps to reduce the complexity of these
two classes that we now have. Changing the way some code is encapsulated might be a great way to hide behavior or make it easier
to use or re-use, but moving this code around has only hidden the complexity in a different place rather than
contributing to its simplicity.

For example, consider a simple method in a Rails app:

```ruby
class PostController < ApplicationController
  def top_comments
    comments = Post.find(post_id).comments.where(
      # complex sql
    )
    render :show_comments, locals: { comments: comments }
  end
end
```

One might stumble upon this in an existing codebase and say, "Let's abstract away that query
logic from the controller." So you may end up with something like this:

```ruby
class Post < ActiveRecord::Base
  def most_recent_upvoted_comments
    comments.where(
      # complex sql
    )
  end
end

class PostController < ApplicationController
  def top_comments
    comments = Post.find(post_id).most_recent_upvoted_comments
    render :show_comments, locals: { comments: comments }
  end
end
```

Voila! We've abstracted our query to the ```most_recent_upvoted_comments``` method. Or, so we think.
Later, someone else comes along and says, "Why does the model have so many queries? Let's abstract the
query logic away from the model." So our query now moves somewhere else:

```ruby
class CommentQuery
  def most_recent_upvoted_comments_for_post(post_id)
    Comment.where(
      # complex sql
    )
  end
end

class PostController < ApplicationController
  def top_comments
    query = CommentQuery.new
    comments = query.most_recent_upvoted_comments_for_post(post_id)
    render :show_comments, locals: { comments: comments }
  end
end
```

So much abstraction going on! This codebase must be awesome!
Well, let's take a look at whether what we're doing is actually abstraction. Have either of these refactorings
simplified the complexity of our software? I don't think so. Rather, this just shifted the encapsulation of this
logic from one class to another. We've simply moved a complex problem&mdash;the conflating of our domain model Comment
with our database and SQL logic&mdash;from place to place. This change in encapsulation might still be beneficial
from an ease of use standpoint, but we haven't solved our complexity problem. The real abstraction to be made
here is separating the *what*&mdash;our domain object, a Comment&mdash;from *how* it is stored and found. In this
specific example, when we're working with ActiveRecord, this combination is just a reality that we've come to accept.

So, what might an actual abstraction around this problem look like? Well, to borrow another heuristic from
Hickey, making things simpler usually necessitates more things&mdash;more classes, or modules, or namespaces that each
do one simple thing. We want to "decomplect" our codebase by abstracting complex operations to smaller, simpler,
more composable pieces. In our example, we can start by separating how a Comment is represented in our code
from how it is stored:

```ruby
class Comment
  attr_reader :post_id :text

  def initialize(attributes)
    @post_id = attributes[:post_id]
    @text = attributes[:text]
  end
end

class CommentRepository
  def most_recent_upvoted(filter = {})
    results = db_connection.select(:post_id, :text)
                           .from(:comments)
                           .where(
      # complex sql
    )
    results.map { |result| Comment.new(result) }
  end
end

class PostController < ApplicationController
  def top_comments
    repository = CommentRepository.new
    comments = repository.most_recent_upvoted(post_id: post_id)
    render :show_comments, locals: { comments: comments }
  end
end
```

Now, we've eliminated the back and forth debate of, "Should this query go in the controller, or the model,
or a query object?" The answer is now, clearly, none of the above. SQL and database logic for a Comment should
be completely disentwined from the rest of our program.

After this abstraction, a Comment can just be a structure for our comment data, our controller can just
ask the correct repository to give it some Comments, and the repository can truly abstract away our query logic.
The repository is an abstraction on top of our data store that the controller can depend on to return
Comment objects&mdash;*what* it needs&mdash;without anything else having to worry about *how* to get Comment objects.
Contrast this with the previous examples, where anything wanting to get Comment objects also had to know
the details of *how* to get Comments. Now, the only way to get *what* we want is to go through the
repository, and the *how* is simply the method defined on the repository. The details of *how* could change
without anything else caring about not being able to get Comments. We've disentangled the two
knotted threads of a controller and model into three straight threads of controller, model, and data storage.

To further demonstrate some abstraction of the PostController, what if it also had a ```create_comment```
method that used the CommentRepository:

```ruby
class Comment
  attr_reader :post_id :text

  def initialize(attributes)
    @post_id = attributes[:post_id]
    @text = attributes[:text]
  end

  def valid?
    text && text.length.between(5, 100)
  end
end

class PostController < ApplicationController
  def create_comment
    commment = Comment.new(comment_params)
    if comment.valid?
      repository = CommentRepository.new
      repository.save_comment(comment)
      redirect_to comment
    else
      render :new, locals: { error: "Invalid comment" }
    end
  end
end
```

We may want to move our specific validation logic into some validation modules to hide the specifics
of any validations on the Comment and to reduce the duplication between other models that use
the same validations.

```ruby
class Comment
  include Validatable
  extend Validations
  validate_length :text, 5, 100

  attr_reader :post_id :text

  def initialize(attributes)
    @post_id = attributes[:post_id]
    @text = attributes[:text]
  end
end
```

This is a great way to encapsulate our common validations. It's really easy to share this
logic between any other models in our code. But, we should be asking ourselves how we
can abstract *why* a Comment may not be valid from the model itself. Again, this requires
more pieces:

```ruby
class Comment
  attr_reader :post_id :text

  def initialize(attributes)
    @post_id = attributes[:post_id]
    @text = attributes[:text]
  end
end

class CommentForm
  def initialize(params)
    @params = params
  end

  def data
    {
      text: params[:text],
      post_id: params[:post_id]
    }
  end

  def valid?
    params[:text] &&
      params[:text].length.between?(5, 10)
  end
end

class PostController < ApplicationController
  def create_comment
    form = CommentForm.new(comment_params)
    if form.valid?
      repository = CommentRepository.new
      comment = repository.create_comment(form.data)
      redirect_to comment
    else
      render :new, locals: { error: "Invalid comment" }
    end
  end
end
```

Now neither the Comment nor the PostController knows *why* a comment is valid or invalid.
The *why* is the responsibility of the form object.
Is this as easy to use as the first example? Probably not&mdash;we have more objects now.
But each piece is much simpler and can be composed in our ```create_comment``` method
in a way that does the same thing in a much less complected way. We could still encapsulate
some shared validations to use in the CommentForm to make validation in other form objects
easier, but it won't be abstracting the *why* any further from the form object.

### Reduce Incidental Complexity

>_Einstein repeatedly argued that there must be simplified explanations of nature, because God is not
capricious or arbitrary. No such faith comforts the software engineer. Much of the complexity he must
master is arbitrary complexity, forced without rhyme or reason by the many human institutions and systems
to which his interfaces must conform. These differ from interface to interface, and from time to time,
not because of necessity but only because they were designed by different people, rather than by God._

>_&mdash; Frederick P. Brooks, The Mythical Man-Month_

Next time you or I start to utter the "a" word, we should stop and think about whether we're talking
about separating our *whats* and *whens* from our *hows* and *whys*, or just shifting the encapsulation
to make incidental complexity easier to manage. As Brooks wrote, we don't have a software god to continuously untangle our
messes, so let's take care to use both of our tools&mdash;abstraction and encapsulation&mdash;appropriately for reducing
complexity when we can, and making things easier while we can't.
