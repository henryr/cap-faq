The CAP FAQ

1. Where did the CAP Theorem come from?

Dr. Eric Brewer gave a keynote speech at the Principles of Distributed
Computing conference in 2000 called 'Towards Robust Distributed Systems'.

[1] http://www.cs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf

2. What does the CAP Theorem actually say?

The CAP Theorem (henceforth 'CAP') says that it is impossible to build
an implementation of read-write storage in an asynchronous network that
satisfies all of the following three properties:

* Availability - will a request made to the data store always eventually complete?
* Consistency - will all executions of reads and writes seen by all nodes be _atomic_ or _linearizably_ consistent?
* Partition tolerance - the network is allowed to drop any messages.

The next few items define some of the terms.

What is 'read-write storage'?

The CAP Theorem specifically concerns itself with a theoretical
construct called a _register_. A register is a data structure with two
operations:

* set(X) sets the value of the register to X
* get() returns the value in the register

3. What does _atomic_ (or _linearizable_) mean?

Atomic, or linearizable, consistency is a guarantee about what values
it's ok to return when a client performs get() operations.

The basic idea is this: consider an execution consisting the total set
of operations performed by all clients, potentially concurrently. The
results of those operations must, under atomic consistency, be
equivalent to a single serial (in order, one after the other)
execution of all operations. The idea is that the register appears as
though it ran on just one machine, and responded to operations in the
order they arrive.

This guarantee is very strong. It rules out, amongst other guarantees,
_eventual consistency_, which allows a delay before a write becomes
visible. So under EC, you might have:

set(10), set(5), get() = 10

But this execution is invalid under atomic consistency.

For more information: [linearizability reference]

When does a system have to give up C or A?

The CAP Theorem only guarantees that there is _some_ circumstance in
which a system must give up either C or A. Let's call that
circumstance a _critical condition_.  The theorem doesn't say anything
about how likely that critical condition is. Both C and A are strong
guarantees: they hold only if 100% of operations meet their
requirements. A single inconsistent read, or unavailable write,
invalidates either C or A.

Since most distributed systems are long running, and may see millions
of requests in their lifetime, the CAP Theorem tells us to be
cautious: there's a good chance that you'll realistically hit one of
these critical conditions, and it's prudent to understand how your
system will fail to meet either C or A.

4. What does _asynchronous_ mean?

An _asynchronous_ network is one in which there is no bound on how
long messages may take to be delivered by the network or processed by
a machine. The important consequence of this property is that there's
no way to distinguish between a machine that has failed, and one whose
messages are getting delayed.

5. What does _available_ mean?

A data store is available if and only if all get and set requests
eventually return a response that's part of their specification. This
does _not_ permit error responses, since a system could be trivially
available by always returning an error.

There is no requirement for a fixed time bound on the response, so the
system can take as long as it likes to process a request. But the
system must eventually respond.

Notice how this is both a strong and a weak requirement. It's strong
because 100% of the requests must return a response (there's no
'degree of availability' here), but weak because the response can take
an unbounded (but finite) amount of time.

6. What is a _partition_?

A partition is when the network fails to deliver some messages to one
or more nodes by losing them (not by delaying them - eventual delivery
is not a partition).

The term is sometimes used to refer to a period during which _no_
messages are delivered between two sets of nodes. This is a more
restrictive failure model. We'll call these kinds of partitions _total
partitions_.

The proof of the CAP Theorem relied on a total partition. In practice,
these are arguably the most likely since all messages may flow through
one component; if that fails then message loss is usually total
between two nodes.

7. Why do some people get annoyed when I characterise my system as CA?

Brewer's keynote, the Gilbert paper, and many other treatments, places
C, A and P on an equal footing as desirable properties of an
implementation. However, this is often considered to be a misleading
presentation, since you cannot build 'partition tolerance'; your
system either might experience partitions or it won't.

The CAP Theorem is better understood as describing the tradeoffs you
have to make when you are building a system that may suffer
partitions. In practice, this is every distributed system: there is no
100% reliable network. So (at least in the distributed context) there
is no realistic CA system. You will potentially suffer partitions,
therefore you must at some point compromise C or A.

Therefore it's arguably more instructive to rewrite the theorem as the
following:

Possibility of Partitions => Not (C and A)

i.e. if your system may experience partitions, you can not always be C
and A.

There are some systems that won't experience partitions - single-site
databases, for example. These systems aren't generally relevant to the
contexts in which CAP is most useful.

8. What about when messages don't get lost?

A perhaps surprising result from the Gilbert paper is that no
implementation of an atomic register in an asynchronous network can be
available at all times, and consistent even when no messages are lost.

This result depends upon the asynchronous network property.

9. Is my network really asynchronous?

Arguably, yes. Different networks have vastly differing characteristics.

If

* Your nodes do not have clocks (unlikely) or they have clocks that may drift apart (more likely)
* System processes may arbitrarily delay delivery of a message (due to retries, or GC pauses)

then your network may be considered _asynchronous_.

10. What, if any, is the relationship between FLP and CAP?

The FLP result is not directly related to CAP, although they are
similar in some respects. Both are impossibility results about
problems that may not be solved in distributed systems. The devil is
in the details. Here are some of the ways in which FLP is different
from CAP:

* FLP permits the possibility of one 'failed' node which is totally
  partitioned from the network and does not have to respond to
  requests.
* Otherwise, FLP does not allow message loss; the network
  is only asynchronous but not lossy.
* FLP deals with _consensus_, which is a similar but different problem
  to _atomic storage_.

11. Are C and A 'spectrums'?

It is possible to relax both consistency and availability guarantees
from the strong requirements that CAP imposes and get useful
systems. In fact, the whole point of the CAP Theorem is that you
_must_ do this, and any system you have designed and built relaxes one
or both of these guarantees. The onus is on you to figure out when,
and how, this occurs.

Real systems choose to relax availability - in the case of systems for
whom consistency is of the utmost importance, like ZooKeeper. Other
systems, like Amazon's Dynamo, relax consistency in order to maintain
high degrees of availability.

12. What's the relationship between CAP and performance?

CAP is

13. What does CAP mean to me as an engineer?

14. Is a failed machine the same as a partitioned one?

No. A 'failed' machine is usually excused the burden of having to
respond to client requests. CAP does not allow any machines to fail
(in that sense it is a strong result, since it shows impossibility
without having any machines fail).

It is possible to prove a similar result about the impossibility of
atomic storage in an asynchronous network when there are up to N-1
failures

16. Have I 'got around' or 'beaten' the CAP theorem?

No. You might have designed a system that is not heavily affected by
it. That's good.


---------------------

What's the relationship between ACID and CAP?
