---
title: Chronic and Acute Problems in Software
---

## Identifying the Illness

Medicine clearly distinguishes between chronic and acute conditions. Acute
conditions have a clear and relatively sudden onset, affect a clearly
defined component of the system, and respond quickly to treatment. Chronic
conditions develop gradually, affect multiple parts of the system, and respond
uncertainly to treatment.

Due to these differences, treatment for acute conditions differs greatly from
treatment for chronic conditions in methods and efficacy. Chronic illness
is the leading cause of death and disability in the US, while medicine has
drastically improved at treating acute conditions. The current success rate for
heart bypass surgery is 95 to 98 percent. Meanwhile, chronic heart disease
remains the leading cause of death in the US. Medicine is good at treating
acute conditions, but due to the more complex nature of chronic conditions, has
not been able to achieve the same level of success with treating chronic
conditions.

Compare this with certain kinds of problems we try to solve as developers,
project managers, and business owners in software projects:

- There's a bug in the ordering system, so we track it down and fix it.
- The business needs a new feature added, so we write a story, and add the
  feature.
- Our response time has degraded, so we do some diagnostics, rewrite a few
  things, and speed it up.
- Our developers have decided that we'd be more productive writing our app in
  React, rather than Angular, so we rewrite it in React.

These are all acute problems. There are clear pain points, symptoms, and
solutions for these problems. We like solving these problems because there's a
sense of urgency, a knowable place to start, and quick feedback once we've
destroyed the sickness. And we're good at solving these problems.

But there are other kinds of problems we encounter in software that lack these
characteristics:

- We've overrun many deadlines, so we try to estimate better, and write
  clearer stories, but we still frequently miss our estimates.
- We chose to rewrite our app, but the development team failed to finish in
  time, now we're scrapping the project.
- The last project failed, so now the business doesn't trust the development
  team, and now developers feel like they're being micromanaged.

These are harder problems to solve. How do we become productive again? How do
we restore the business's trust in the engineering team? How do we ensure
business owners are clearly communicating feature requirements to
developers? These problems are more akin to chronic conditions, affecting
multiple parts of our team in different ways, with an uncertain root cause, and
developing gradually.

## A Systems Perspective

We can't treat these problems the same way we pursued our acute problems.
Rewriting our Ruby app in Go is not going to prevent the dev team from ever
missing another deadline. Switching from incremental to fibonacci estimates
isn't going to make you and the business argue about estimates less frequently.
Using Jira instead of Pivotal Tracker isn't going to better prioritize your
work. Asking your Ops team to be faster is not going to get you that database
sooner. Asking the business owners to prioritize giving you information on how
to implement the next feature over planning and rehashing the entirety of
the project isn't going to get the project done sooner.

These kinds of chronic problems stem from conditions that
exist across the system and manifest themselves in specific ways, some of
which we end up seeing firsthand. To solve chronic problems on our
software teams, we need to approach these problems from a systems perspective.

As Donella H. Meadows explains in "Thinking in Systems," a system is "an
interconnected set of elements that is coherently organized in a way that
achieves something." The connections and flows in a system manifest as stocks.
Stocks are things that we can see, feel, and measure. On our teams we have
stocks of things like knowledge, technical debt, pressure, money, time,
happiness, and power at different places throughout our organization. Changes
to one stock create changes in other stocks through various feedback loops that
act based on a set of decisions, rules, or physical laws.

For example, you have a feedback loop that assesses your job satisfaction. The
flows of both pay and workload into your stocks of money and stress initiate
this loop. When your stress stock changes, the feedback loop involves you
reassessing your job satisfaction. You might meet with your employer to discuss
changing the flow of pay to your money stock or workload affecting your stress
stock.

![money system](/assets/money.png)

Of course, systems are more complex than we can model in one diagram. This is a
simplification of the system focusing only on the two things we're discussing
in this scenario, as any diagram of any system would be a simplification. The
important part of a system diagram like this is to identify the components
of our system relevant to the problem we are trying to solve.

## Human Software Systems

Once we start identifying the flows, stocks, and feedback loops of the
system facing a chronic problem, we can then figure out where to add, remove,
or modify certain feedback loops to bring our stores into proper levels
over time.

For example, if we need to build more knowledge of business requirements
in our dev team's store of knowledge, we need to build up the feedback loop
that asks for more information and adjusts that flow. That probably sounds
simpler than it is though, because it requires effort from multiple parts of
our team&mdash;developers taking the time to ask questions and business owners
being available and taking the time to answer questions (and vice-versa).
It may involve adding new people to the team to facilitate this loop, or
maybe moving others to new roles if their current position inhibits the
feedback loop.

Problems of trust&mdash;territorialism; animosity between the dev team, project
managers, and business owners; aversion to change; excessive planning and
process&mdash;often arise due to a system condition that Meadows calls "seeking the
wrong goal." In these systems, "behavior is particularly sensitive to the
goals of feedback loops. If the goals&mdash;the indicators of satisfaction of the
rules&mdash;are defined inaccurately or incompletely, the system may obediently
work to produce a result that is not really intended or wanted."

In this case, developers may have a feedback loop that tells them to continue
writing code, but no feedback loop exists to tell them whether the code they're
writing is buggy or whether people actually like what they're building. Managers may
have a feedback loop that determines whether they've had the right
meetings, but no feedback loop that tells them what pain points their reports
would actually like to discuss. Business owners may have a feedback loop that
tells them whether they've outlined everything they think the developers
should know about their needs, but no feedback loop that adequately tells them
what the developers really need to know *right now*.

![feedback loop 1](/assets/loop1.png)

In this system, everyone has a feedback loop that seeks to satisfy their own
goals and the system ends up slowly producing some software that works, but
there are lots of bugs and nobody is particularly happy. The system is missing
feedback loops that address the real welfare of the system&mdash;that of creating
high-quality software that the business loves to use.

![feedback loop 2](/assets/loop2.png)

XP, Agile, Kanban, all strive to explicitly drive out these important feedback
loops and turn them into habits. What you call the process doesn't actually
matter once you can clearly say that your process provides the necessary
feedback loops for your system to seek the right goal.

## Changing Habit

The hardest part of fixing any of these chronic, system-wide problems is that
they involve changing people's habits. Changing habit is hard, specifically because
the entire point of a habit is to allow us to reduce the amount of effort we put
forward in doing certain things:

>_When a habit emerges, the brain stops fully participating in decision making.
It stops working so hard, or diverts focus to other tasks. So unless you
deliberately fight a habit—unless you find new routines—the pattern will unfold
automatically._

>_&mdash; Charles Duhigg, The Power of Habit_

In fact, you could say that organization-wide problems in software resemble
chronic illness because they are brought about by habit, which itself resembles
a chronic illness. Habit, too, develops gradually, is long-lasting, and
responds uncertainly to treatment. But, we know we *can* change our habits. It
just takes time and awareness of the habit and the conditions that led to it.

Software developers and participants in the software development process love
to solve problems. But we can't approach every problem the same way. Our
biggest failure in solving certain chronic problems would be to look for clear
root causes and expect quick results once our clear-cut solution is
implemented&mdash;as if we can solve these complex problems the same way we've
solved acute problems on our teams. By acknowledging the complexity of chronic
problems, we can give ourselves and our team members a more accurate assessment
of the multi-faceted, long-term, uncertain, and habit-changing approach that we
need to take to solve the problem. That way we can approach the problem with
the tact and patience it deserves while reducing discouragement and frustration
for ourselves and our team members when our simple solutions don't fix things
right away.

## References
- [CDC](http://www.cdc.gov/chronicdisease/overview/index.htm)
- [Thinking In Systems](http://www.chelseagreen.com/thinking-in-systems)
- [The Power Of Habit](http://charlesduhigg.com/the-power-of-habit/)
