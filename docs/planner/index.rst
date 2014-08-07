=========
 Planner
=========

High level description
======================

The planner figures out how many machines there ought to be right now,
and then communicates that amount to the converger.

It is functionally pure, only violating that constraint when acquiring
input and producing output:

- As input, it takes the current set of policies defined by the user,
  as well as some context about the world, such as the current time.
- As output, it notifies the converger.

Internals
=========

(This space is intentionally left blank. I feel like the answer here
is "constraint logic programming", and have attempted to explain why
in a separate e-mail thread. I haven't elaborated on that here in case
people vehemently disagree with me that the answer is constraint
programming.)
