Conflict-free Replicated Data Types
<https://pages.lip6.fr/Marc.Shapiro/>
https://gist.github.com/russelldb/f92f44bdfb619e089a4d  - looks good
https://medium.com/@istanbul_techie/a-look-at-conflict-free-replicated-data-types-crdt-221a5f629e7e
https://vaughnvernon.co/?p=1012
http://christophermeiklejohn.com/crdt/2014/07/22/readings-in-crdts.html
https://gist.github.com/haadcode/7ed13d6c34696652a802
https://github.com/ljwagerfield/crdt
https://syncfree.lip6.fr/index.php/crdt-resources
https://www.slideshare.net/InfoQ/practical-data-synchronization-using-crdts
https://irisate.com/crdt-for-real-time-collaborative-apps/
https://redislabs.com/blog/getting-started-active-active-geo-distribution-redis-applications-crdt-conflict-free-replicated-data-types/?utm_content=buffer9cfba&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer

https://www.youtube.com/watch?v=em9zLzM8O7c - soundcloud crdts

long thesis to read:
https://hal.inria.fr/inria-00555588/PDF/techreport.pdf

read this: @phdthesis{shapiro2011comprehensive,

strong eventual consistency and conflict-free replicated data types youtube vid

watch: https://www.youtube.com/watch?v=ebWVLVhiaiY
maybe watch: https://www.youtube.com/watch?v=yCcWpzY8dIA


watched: https://www.youtube.com/watch?v=vBU70EjwGfw
  seems good, has versioning, vector clocks info, convergence
also see: https://serverless.com/blog/crdt-explained-supercharge-serverless-at-edge/

What
Why
How

What
----
<https://www.youtube.com/watch?v=9xFfOhasiOE>

In a simple model, client server model, you have one server that is
read from and written to by many clients.  However, the single server
is neither fault tolerant nor optimized for local performance if
clients are widely separated nor able to scale a large load beyond
that of the single server limitations.

So we distribute the servers so that each can be read from and
written to and find a way to have them appear as a consistent
store.  This is difficult and impossible when there are failures
(i.e. what if two clients write to the different servers during
a partition event).

When there are partitions you cannot have availability and
consistency at the same time.  When the failures subside, how do
you form a consensus?  Enter eventual consistency.  Eventually
the system should converge, but this is a complex problem.  CRDTs
have arise to help address this problem.

** when two nodes see the same events, they should immediately be
in the same (consistent) state.  This raises the bar, makes it a
strong eventually consistency, but makes the algorithm more simple**
  - why?


operation-based versus state-based counters
  operation-based must "commute", for example:

    (x-4)-3 = (x-3)-4
  operation-based must apply only once, for example:

    do not duplicate packets

conflict-free doesn't necessary mean exactly what one process may
have wanted, may get a different result depending on timing, order,
or ids of processes.  for example, if two processes add a value to
a set, but one is delivered very slowly.  In the meantime, the other
removes that item from the set before the slow one reached it.  When
the slow add is finally delivered, will there be an inconsistent
state? no, you label each one from each process, and the odd/high/low
id one wins. This ensures the end result is conflict-free, thought
it may provide a "surprising" result in some cases or to some nodes.

3 rules for operation-based:
  * all (concurrent) operations must commute
  * require exactly-once delivery semantics
  * require in-order delivery


state-based, instead of sending the operation to other nodes, you do
the operation locally then send your state over the network, you then
need to merge the states


go through types
example slide

introduction
example types
  counters, sets
  covergent vs commutative
psuedo code
