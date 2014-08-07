===========
 Converger
===========

High level description
======================

The converger gets told what amount of machines any given load
balancer should have, and then does whatever it has to in order to
maintain that amount.

It attempts to not overload the systems it operates on (such as Nova
and Cloud Load Balancers) through the use of a Kalman filter. The
converger is adaptive over time with respect to other systems.

Black-box sequence diagrams
---------------------------

Key:

- Tenants are represented by Greek letters (α, β, ...)
- Load balancers are represented by capital Latin letters (A, B, ...)
- Servers are represented by small Latin letters (a, b, ...)

Example 1
~~~~~~~~~

.. seqdiag:: diagrams/1/start.diag

At some point in the future, we get a Nova event that a server is up,
telling us the server id and the tenant id. The implementation defines
some way to translate that server id to the target cloud loud balancer
id; presumably storing that in a database somewhere.

.. seqdiag:: diagrams/1/nova_event.diag

The converger sees the new server, *d*, and adds it to the CLB, A.
Because the remaining pending server was requested sufficiently early,
it doesn't decide to ask for another server just yet. (This is the
Kalman filter behavior described below.)

Internals
=========

Data model
----------

.. blockdiag:: diagrams/model.diag

Note the absence of policies: old-style policies, new-style policies
(see planner) and min-max constraints.

Kalman filters
--------------

Otter could potentially ask things of Cloud Load Balancers and Nova
that make those services (and their dependent services) very unhappy,
such as asking for too many servers and then changing its mind.

In essence, a Kalman filter is simply a *stateful* filter, that is:
the data that it throws out depends on the data that it has previously
thrown out.

In our case, the input and outputs are discrete: the input is how many
servers Nova knows about, the output is how many servers to ask for.

.. blockdiag::

   default_fontsize = 14;

   in [shape=beginpoint, label=""];
   out [shape=endpoint, label=""];
   filter [label="K"];
   in -> filter -> out;

As an early prototype, it does this by talking to Nova at most once in
a (variable) amount of time.

This can be made more sophisticated at a later date, as necessary.

Workers and work distribution
-----------------------------

The converger consists of one or more workers, consuming data from a
queue. This queue provides at least a "soft" guarantee ("it is very
unlikely that...") that only one worker will be assigned a particular
load balancer. This is okay because more than one convergence taking
place at the same time simply means that we'll overcompensate; the
next convergence action (which typically occurs quite soon after any
attempt to add servers) rectifies this.

Future work
===========

Limiting impersonation tokens
-----------------------------

Otter relies on creating impersonation tokens. These are limited-use,
but it would still be nice if such a high-authority capability (the
ability to produce such impersonation tokens) was encapsulated in a
smaller, easier-to-audit component.
