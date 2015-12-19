---
title: "Code Smells"
---

I’ve been reading Agile Software Development: Principles, Patterns, and
Practices, by Bob Martin, recently. One part I’ve really enjoyed is his
discussion of “design smells - the odors of rotting software.” It’s interesting
to hear about what causes software projects to go wrong, and think about how I
can write code that avoids these problems. Bob talks about 7 problems that
exhibit “rotting” software.

1. Rigidity - The system is hard to change because every change forces many
other changes to other parts of the system.

2. Fragility - Changes cause the system to break in places that have no
conceptual relationship to the part that was changed.

3. Immobility - It is hard to disentangle the system into components that can
be reused in other systems.

4. Viscosity - Doing things right is harder than doing things wrong.

5. Needless Complexity - The design contains infrastructure that adds no direct
benefit.

6. Needless Repetition - The design contains repeating structures that could be
unified under a single abstraction.

7. Opacity - It is hard to read and understand. It does not express its intent
well.

From the projects I’ve worked on, I can easily see how adding more of any of
these factors would make the code unfriendly to work with. I’ve also seen how
keeping the SOLID principles in mind can help you avoid these code smells.

Another interesting part of this book so far is seeing how writing good code
that’s easy to change and utilizes the SOLID design principles is so closely
tied to being able to practice Agile software development effectively. I
previously hadn’t made that connection between the short iterations and
openness to changing requirements, and the need for code that is modular and
easy to change. I had only seen good code design as a benefit to the actual act
of writing code, but it’s also a huge benefit to the Agile business process.

I’ve really enjoyed this book so far. I think it has a lot of highly applicable
material, so I’m looking forward to learning a lot as I finish reading.
