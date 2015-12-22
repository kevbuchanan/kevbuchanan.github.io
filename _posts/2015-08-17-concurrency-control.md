---
title: Concurrency Control Strategies for Secret Agents
---

In the 1963 movie "Dr. No," the first in the James Bond series, Bond travels to
Jamaica for a case, and he knows that his enemies are watching his every move.
He knows his enemies are smart, but he's smarter. To keep them from prying
into his plans, he can't simply lock them out of his room, he needs to be a
little more sly&mdash;as any reputable villain could surely pick a lock. So he
places a hair on the door frame, knowing that in the sure event that his room
is infiltrated while he's out, he'll see that the hair has moved and can
abondon or change his plan if needed. It's a brilliant tactic, and saves
him time, effort, and constraints on his travel that he'd incur by trying
to securely lock down his room as he moves from place to place.

How would this scenario look in code? Well, let's introduce a secret mission,
in Ruby:

```ruby
class SecretMission
  def initialize
    @files = Array.new(5) { |i| "File #{i + 1}" }
  end

  def read(name)
    while file = @files.shift
      puts "#{name} is reading #{file}"
      sleep(0.1)
    end
  end

  def accept
    puts "Mission Accepted"
  end
end
```

The mission will contain some files that need to be read. Bond
will accept the mission once he reads the files. But, there are also some
villains out there trying to read the files.

```ruby
mission = SecretMission.new
villains = ["Odd Job", "Jaws", "May Day"]
threads = []

threads << Thread.new do
  mission.read("Bond")
  mission.accept
end

villains.map do |spy|
  threads << Thread.new do
    mission.read(spy) if rand(0..3) == 0
  end
end

threads.map(&:join)
```

With everything running in a separate thread, we get something like this output:

```
Bond is reading File 1
Odd Job is reading File 2
Jaws is reading File 3
Bond is reading File 4
Jaws is reading File 5
Mission Accepted
```

Since there's no coordination of who is reading the files, a couple
spies can sneak in. This is bad news. Bond just accepted a doomed mission
because he didn't realize that his plans were being read by the villains
at the same time.

Maybe we can make him a little safer if we lock down the files when
they're being read. We could add a mutex to the `SecretMission` class
to use when reading the files:

```ruby
class SecretMission
  def initialize
    @files = Array.new(5) { |i| "File #{i + 1}" }
    @mutex = Mutex.new
  end

  def read(name)
    until @mutex.try_lock
      puts "#{name} is waiting"
      sleep(0.3)
    end
    puts "The files are gone!" if @files.empty?
    while file = @files.shift
      puts "#{name} is reading #{file}"
      sleep(0.1)
    end
    @mutex.unlock
  end

  def accept
    puts "Mission Accepted"
  end
end
```

Now when we run our spies in separate threads, we consistently see something
like this (assuming Bond's thread is the first to read):

```
Bond is reading File 1
May Day is waiting
Bond is reading File 2
Bond is reading File 3
May Day is waiting
Bond is reading File 4
Bond is reading File 5
Mission Accepted
The files are gone!
```

Bond gets to the files, acquires the lock, and the other threads are locked
out for the duration of his reading. But, let's assume that using this lock is
time consuming and really draining the MI6 budget. Furthermore, once Bond has
the lock, the other spies have to wait around for the lock to be available, and
they just cause trouble.

So let's try to emulate Bond's hair on the door trick and allow us to check
whether any villains have caught up to him so that we can then abort the mission
and throw them off of his tracks.

First, let's change our `read` method to return how many files are read, and change
the `accept` method to take a parameter for how many files a spy was able to read.

```ruby
class SecretMission
  def initialize
    @files = Array.new(5) { |i| "File #{i + 1}" }
  end

  def read(name)
    puts "The files are gone!" if @files.empty?
    read = 0
    while file = @files.shift
      puts "#{name} is reading #{file}"
      sleep(0.1)
      read += 1
    end
    read
  end

  def accept(read)
    if read == 5
      puts "Mission Accepted"
    else
      puts "Abort Mission!"
    end
  end
end
```

Then, let's change Bond's behavior so that he can signal how many files
he got to:

```ruby
threads << Thread.new do
  files_read = mission.read("Bond")
  mission.accept(files_read)
end
```

Now, we could see Bond read all of the files successfully if none of the
villains find them, or we could see something like this:

```
Bond is reading File 1
Odd Job is reading File 2
Odd Job is reading File 4
Bond is reading File 3
Odd Job is reading File 5
Abort Mission!
```

This is a nice, lightweight option that allows us to always know whether
the mission is compromised, without any waiting by any of the spies.

## From Pessimistic to Optimistic Locking

What we've done here is moved the concurrency control mechanism from
a pessimistic lock using a mutex, to optimistic concurrency control, which
assumes that nobody else is reading our files&mdash;until we verify that assumption
at the end of our transaction. If we were using a database to read and write
our data, we could use a pessimistic lock by obtaining a lock for a specific
row in the database.

Let's use another example, in which we have orders that we want to cancel. We
can only cancel orders if they are currently in a "pending" state. It also
causes a lot of problems to cancel an order twice, or an order that is not pending.
But we have lots of updates happening concurrently, so we need some way to maintain the
integrity of our concurrent state transitions. We could start with a
pessimistic lock. We'll assume we're using ActiveRecord for this example.

```ruby
order = Order.find(order_id)
order.with_lock do
  if order.status == 'pending'
    order.update({ status: 'cancelled' })
  end
end
```

If we're using PostgreSQL as our database, this will perform a `SELECT FOR UPDATE`
on the record prior to yielding to the block to perform. Once this transaction gains
the lock through the `SELECT FOR UPDATE`, the database will not allow other
transactions to modify the row until our transaction is committed, after the block
finishes. This gives us assurance that nothing else will change the order while the current
thread is using it. But sometimes this is a heavy tool for the job. For one,
any other threads trying to use the row will block, which could degrade our
performance.

We could avoid the need to acquire a lock, and wait for the lock, by changing
this procedure to an optimistic lock. All we need to do is tell the database
the state of the object that we have, and let it confirm that that
is indeed the current state. If it is not the current state, we know that
another transaction was working at the same time. We can do this by using an
`UPDATE WHERE`. In ActiveRecord that would look like this:

```ruby
order = Order.find(25)

if order.status == 'pending'
  rows_affected = Order.where({
    id: order_id,
    status: 'pending'
  }).update_all({ status: 'cancelled' })

  raise StaleDataError if rows_affected == 0
end

```

The above query should generate the SQL:

`UPDATE orders SET orders.status='cancelled' WHERE orders.id=25 AND orders.status='pending';`

A SQL update returns the number of rows that were updated, so we can check the return
value from our update and know that we've run into concurrent updates if our update
returns `0`.

*Note: If you're using ActiveRecord, or another ORM to do a similar optimistic lock,
you should confirm that this is actually the SQL that gets generated. I've seen
ActiveRecord convert queries like this to a sub-select, which negates the
benefit of our lock because we're left with a race condition between
when the sub-select executes and when other concurrent updates commit.*

## Multi-Version Concurrency Control

### PostgreSQL

Although our optimistic locking strategy isn't explicity acquiring a lock, we could
still block other transactions. Certain databases might block both reads and writes
while we are performing the update. PostgreSQL will only block other writes.
When using an optimistic locking strategy in PostgreSQL, it's important to
know why this works, and how it affects other transactions. In order to perform
our `UPDATE WHERE` while maintaining our data consistency and also not blocking
other threads from reading data, PostgreSQL uses a technique called Multi-Version
Concurrency Control (MVCC). Other databases also use this pattern, but I'll
focus on PostgreSQL here.

PostgreSQL executes each statement inside of a transaction with the *Read
Committed* isolation level. An isolation level determines what kinds of changes
to data are shared between transactions. There are other isolation levels
available, but *Read Committed* is the default. In *Read Committed* isolation,
PostgreSQL allows other threads to read the row without blocking, though other
updates will need to wait for other concurrent updates to complete. How does
this work? Well, the key here is that each record in the database isn't just
one row. It's actually many rows that PostgreSQL is constantly marking as
unavailable after updates or deletes, and available as they become the most
recent version (it's actually a slightly more complicated process of comparing
transaction IDs stored in [system columns](http://www.postgresql.org/docs/9.4/static/ddl-system-columns.html),
but let's use this simplification). PostgreSQL eventually goes back and cleans
up the unavailable rows through the
[*autovacuum*](http://www.postgresql.org/docs/9.4/static/routine-vacuuming.html#AUTOVACUUM) process.

So, back to our example, say we execute our query at time `T1`:

`BEGIN; UPDATE orders SET orders.status='cancelled' WHERE orders.id=25 AND orders.status='pending'; COMMIT;`

PostgreSQL begins a transaction in *Read Committed* isolation, and for the
duration of this transaction it will only see data committed and marked as
available prior to `T1`. Then, say we execute a concurrent query at time `T2`
to:

`BEGIN; SELECT status FROM orders WHERE orders.id=25; COMMIT;`

This `SELECT` will only see row versions that were committed and available
prior to `T2`. Assuming we only have these two queries running, the `SELECT`
will see the same version of the row that our first `UPDATE` is
seeing&mdash;that which was committed prior to `T1`, with the status as
"pending". So this query will return "pending". (Side note: a `SELECT` will
also see data changed by an `UPDATE` within its own transaction).

Let's throw in another update and see what happens. Assume our `SELECT` completed
at time `T3` and returned "pending", but our `UPDATE` is still running. Another
transaction might start at time `T4` to actually start the order:

`BEGIN; UPDATE orders SET orders.status='started' WHERE orders.id=25 AND orders.status='pending'; COMMIT;`

This transaction will see the version of the data committed prior to `T1` and our first
update, so it will find a row with ID 25 and status "pending". But it will see that the
row is currently held by a *ROW EXCLUSIVE* lock in our first transaction, so it will
need to wait. Let's assume the first update commits at `T5`. Our second update now
gets to re-apply its query to the row version as of `T5`, so it will see that the status
is now not equal to "pending", and it will not update the status to "started".

So, we're still eventually locking a row in our database, but the benefit of
doing this through PostgreSQL's MVCC implementation is that we minimize the
amount of time that a lock is held, and the amount of time that our threads
spend waiting to proceed with our business logic. This could be good or bad
depending on how our program is structured.

If we perform some side effect-y things prior to updating the order, such as
sending a notification that the order is started, then we've actually been
notified about the concurrent update too late. On the other hand, if we do the
update before the business logic, then we know right away whether we can
proceed and avoid blocking while another thread performs its business logic.
When we use `SELECT FOR UPDATE` by using `order.with_lock` in ActiveRecord,
other threads will halt on trying to acquire the lock while the first executes
the in-memory logic and the database update. In many situations, but not all,
it's more effecient to be optimistic and let things proceed until we know that
we've actually encountered a concurrency issue.

### Clojure STM

Let's take this full circle and see what a MVCC implementation of our secret file
example would look like. For an in-memory implementation, we can use Clojure, as Clojure's
refs are implemented using Software Transactional Memory (STM) with MVCC.

```clojure
(ns spies.core)

(def files
  (->> (range 5)
       (mapv #(format "File %s" (inc %)))
       ref))

(defn read-file [spy files file]
  (println (format "\n%s is attempting to read %s" spy file))
  (alter files rest)
  (println (format "\n%s is reading %s" spy file))
  (Thread/sleep 300)
  file)

(defn accept-mission [files]
  (alter files (constantly ["A bad pun from Bond"]))
  (println "\nMission Accepted"))

(def bond
  (future
    (dosync
      (let [file-read (map #(read-file "Bond" files %) @files)]
        (if (= 5 (count files-read))
          (accept-mission files)
          (println "\nAbort Mission"))))))

(defn villainize [villain]
  (future
    (when (zero? (rand-int 3))
      (dosync
        (if-let [file (first @files)]
          (read-file villain files file)
          (println (format "%s, the files are gone" villain)))))))

(def villains
  (map villainize ["Odd Job" "Jaws" "May Day"]))

(defn -main [& args]
  (let [futures (conj villains bond)]
    (doall (map deref futures)))
  (shutdown-agents))
```

In this code, we create a group of threads to read the files using `future`.
Bond's thread will try to read all the files and then leave a note for
the villains, whereas the villains will only try to read one file. The files
are kept in a `ref`, which will only allow mutation while inside a transaction
using `dosync`. While in the transaction, we can change the ref by using
`alter`, which takes as arguments a ref and a function to call to alter the
ref. When we run this, we see something like this:

```
Bond is attempting to read File 1
Bond is reading File 1
Jaws is attempting to read File 1
Odd Job is attempting to read File 1
Bond is attempting to read File 2
Bond is reading File 2
Odd Job is attempting to read File 1
Bond is attempting to read File 3
Jaws is attempting to read File 1
Bond is reading File 3
Bond is attempting to read File 4
Bond is reading File 4
Bond is attempting to read File 5
Bond is reading File 5
Odd Job is attempting to read File 1
Jaws is attempting to read File 1
Mission Accepted
Jaws is attempting to read A bad pun from Bond
Odd Job is attempting to read A bad pun from Bond
Jaws is reading A bad pun from Bond
Odd Job, the files are gone
Jaws, the files are gone
```

You'll notice that the threads trying to read for Jaws and Odd Job retried a
number of times while Bond's thread was reading the files. This is caused by
Clojure's LockingTransaction implementation retrying the call to `alter` if the
transaction can't obtain a write lock or if it finds that the ref has been
altered since its read point. No threads are blocked from reading the files, but
once a thread makes a call to `alter`, it tries to acquire a write lock, and if
it can't, it will retry from the beginning of the transaction starting from
`dosync`.

Each thread sees the last committed version of the files when the transaction started.
So to start, each villain thread will see that there are five files in the ref,
and will try to remove one to be read by the spy by calling `alter`. However,
Bond's transaction has the write lock, so the other transactions will retry the
`alter` of the first file a number of times, until Bond's transaction commits and
releases the write lock. Then the villain threads will retry again because
Clojure's STM detects that the ref has changed. This time they see the version
of the ref that Bond committed, with only his note left.

## Tradeoffs

Given your options for concurrency control strategies&mdash;pessimistic/locking,
optimistic/lock-free, or MVCC/hybrid&mdash;which one should you use? Well,
like most decisions when writing software, it comes down to your needs for the
specific problem you're trying to solve. Each of these strategies present
different tradeoffs that should be taken into account. To sum up, here's what
you get with each:

#### Pessimistic/Locking

**Benefits:**
- Exclusive access
- Possible stronger guarantee of integrity

**Drawbacks:**

- Lock contention
- Records are possibly completely inaccesible by other processes depending on
  implementation

#### Optimistic/Lock-Free

**Benefits:**

- Lower overhead
- No waiting or lock contention

**Drawbacks:**

- Deciding how to handle collisions or stale data
- Possibility of retrying operations with side-effects

#### MVCC/Hybrid

**Benefits:**

- Reads are never blocked
- Reads never block writes

**Drawbacks:**

- Cost of maintaining multiple versions of records (e.g. autovacuum)
- Handling retried transactions or stale data
- Isolation levels vary between implementations

When trying to solve a concurrency problem, keep in mind that there are many
strategies available to you, and pick the one that best suits your specific
needs. You don't always need the brute force of a pessimistic lock. Bond knew
the villians would break in, he just wouldn't know when or what they had access
to without the hair. You too can embrace the challenges of concurrency through
different concurrency control strategies. With the right choice, you can thwart
would-be concurrent invasions into your process just like a secret agent.

#### References

- [PostgreSQL](http://www.postgresql.org/docs/9.4/static/mvcc.html)
- [PostgreSQL Anti-Patterns](http://blog.2ndquadrant.com/postgresql-anti-patterns-read-modify-write-cycles/)
- [PostgreSQL Versioning and Bloat](http://www.keithf4.com/checking-for-postgresql-bloat/)
- [Testing the I in ACID](http://martin.kleppmann.com/2014/11/25/hermitage-testing-the-i-in-acid.html)
- [Clojure Refs](http://clojure.org/refs)
- [Jay Fields on Clojure Refs](http://blog.jayfields.com/2011/04/clojure-state-management.html)
