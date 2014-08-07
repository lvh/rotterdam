===================
 Otter +2 overview
===================

.. blockdiag:: diagrams/overview.diag

The two main components that remain are:

1. The planner (symbol: Σ, because it's an aggregate function) which
   computes the desired capacity.
2. The converger, which does whatever it has to do to make that
   desired capacity.

(Since there are several definitions floating around: whenever this
document says "desired capacity", it means "the capacity that a
scaling group ought to have".)

These two components have very interesting properties:

- The planner is *pure*: its only observable side effect is that it
  tells the converger what the desired capacity is. It only violates
  referential transparency when acquiring its inputs: on one hand, the
  policies, which the user of course gets to change; on the other
  hand, context such as the current time.
- The converger is *idempotent*: running it several times will produce
  the same result. This means that it's safe to run repeatedly; worst
  case scenario, you waste some CPU cycles.

These components are explained in more detail in the following
sections.
