---
title: "Package Cohesion and Coupling"
---

In preparing a presentation I’ll be giving this week on the Principles of
Package Design that Uncle Bob discusses in Agile Software Development:
Principles, Patterns, and Practices, I’ve chosen to focus on the two main ideas
behind the 6 principles: package cohesion and package coupling. As far as
defining what a “package” is, I’ve come to the conclusion that it’s best to
leave that loosely defined as a collection of code that shares a set of
responsibilities. I think there is value in applying the principles of package
cohesion and coupling regardless of where you draw the line on what constitutes
a package.

>_Package Cohesion addresses how to partition and compose a package to best
balance reusability and maintainability._

When delimiting a package, there are competing forces in the developer wanting
to group things together for convenience, and the user wanting to be isolated
from changes to components that he or she is not using. Packages should be
composed in large enough groups that it’s convenient to use and maintain,
according to what “can’t” be used separately and is affected by the same
changes, but small enough to minimize violation of the Interface Segregation
Principle. Neither of these can be completely satisfied, but if you frame the
packaging in terms of creating package cohesion, you can determine where to
make trade offs between how the code is reused and how the code is maintained.

Package Coupling addresses how to structure relationships between packages such
that instable packages are easy to change, and stable packages are easy to
extend.

When coupling packages, the dependencies should flow from least stable to most
stable, meaning that packages that are intended to change frequently should
depend on packages that are not intended to change. Stability is imposed on
packages by depending on them because they are then responsible to their
dependents, so change becomes harder. Stable packages should thereby be more
abstract in order to be more flexible and extensible to their many dependents.
This can be looked at as the Dependency Inversion Principle and Open/Closed
Principle on a more macro level. We want to depend on abstractions, not on
concretions, and leave our dependencies open for extension, given that they are
stable and closed to modification.

While more details can be found by diving into the 6 package principles
individually, I find it most helpful to think in terms of general package
cohesion and package coupling. Hopefully I’ve interpreted these concepts
correctly and coherently, because I’ve found them to be a great way of
approaching SOLID design from a broader perspective, in terms of whatever your
software “packages” are determined to be.
