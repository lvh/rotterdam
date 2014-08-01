===========
 Converger
===========

High level description
======================

The converger gets told what amount of machines any given load
balancer should have, and then does whatever it has to in order to
maintain that amount.

The converger attempts to not overload the systems it depends on
(Nova, Cloud Load Balancer) by applying a Kalman filter. The converger
is adaptive with respect to other systems over time.

Black-box sequence diagrams
---------------------------

Example 1
~~~~~~~~~

.. seqdiag:: diagrams/1/start.diag

At some point in the future, we get a Nova event. The server id in
this Nova event is translated to the cloud load balancer id.

.. seqdiag:: diagrams/1/nova_event.diag

Internals
=========

Data model
----------

Scaling group consists of:

 - Load balancer
 - Desired capacity
 - Launch configuration

Notably, it does not consist of:

 - Old-style policies
 - Min/max constraints

Kalman filters
--------------

As an early prototype, it does this by talking to Nova at most once in
a (variable) amount of time. This can be made more sophisticated at a
later date, as necessary.

Workers and work distribution
-----------------------------

The converger consists of one or more workers, consuming data from a
queue. This queue provides at least a "soft" guarantee ("it is very
unlikely that...") that only one worker will be assigned a particular
load balancer. This is okay because more than one convergence taking
place at the same time simply means that we'll overcompensate; the
next convergence action (which typically occurs quite soon after any
attempt to add servers) rectifies this.
