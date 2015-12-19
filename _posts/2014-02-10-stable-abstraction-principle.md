---
title: "The Stable-Abstraction Principle"
---

Another section from Agile Software Development: Principles, Patterns, and
Practices that I have found interesting is the Stable-Abstraction Principle.
The SAP states:

A package should be as abstract as it is stable.

A package can be said to be more stable as more packages depend on it. As it
becomes more responsible to other packages, it becomes harder to change, so, it
should be made abstract so that it can be extended when necessary. A package
that is not used by other packages is said to be independent and can be changed
easily without effecting other packages, so it can remain concrete.

Previously, I hadn’t thought about the connection between dependencies and
abstraction, but I think this is a really useful idea in package design. It
becomes very apparent how stability and abstraction relate to one another in
Bob’s graph:

![abstraction graph](http://38.media.tumblr.com/815959353cc284540c3caf96b6edc221/tumblr_inline_n0tbp3SPn21rkh8q7.png)

As abstractness and instability increase, you eventually end up in the “zone of
uselessness” in the top right corner. This class is obviously useless because
it has no dependents and is maximally abstract. Perhaps more commonly seen is
the “zone of pain” in the bottom left corner. Bob uses the example of a
database schema, which is very concrete, but also very stable because much of
app is dependent upon it, which is why database migrations are relatively
painful to do. The ideal place is the black line shown in the graph, called the
“main sequence”. Packages on the main sequence have the right balance of
abstractness and stability.

The entire section on package design has been really interesting to me. Bob
provides a lot of instruction on how to think about dependencies, coupling, and
abstraction when designing packages, so I’ll be revisiting this section when it
comes time to think about the design of any packages I’m building.
