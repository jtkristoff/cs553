# UIC CS 553 - Fall 2017

[Course Home Page](https://www.cs.uic.edu/~ajayk/c553fa17.html)

The class presentation was on CRDTs and can be found in the
corresponding subdirectory.  The term paper topic was
peer-to-peer DNS which can be found in the report subdirectory.

Lecture and book summary as well as study notes for exams to follow.

For each topic, identify the ''what'', ''why'', and ''how''.   That is:

* *What:* A summary of topic, idea, or algorithm.  Some assumptions may be made, but others may need to be identified.
* 'Why:* Make the argument for the reason to study this item.  Why is or was it important in the development of this field of science.
* *How:* If this is an algorithm or technology, explain how this works.

# Introduction - Chapter 1

A distributed system is characterized by the following properties:

  * No common physical clock
  * No shared memory
  * Geographic separation
  * Autonomy and hetergeneity

A distributed system is motivated by the following requirements:

  * Inherently distributed
  * Resource sharing
  * Access to geographically remote data and resources
  * Enhanced reliability
  * Increased performance/cost ratio

A distributed system also has the following advantages:

  * Scalability
  * Modularity and incremental expandability

A parallel/multi-procesor may be:

  1. a system in which multiple processors have direct access to shared memory.
  2. a multicomputer parallel system does not have direct access to shared memory.
  3. array processors that may have a common clock but may not share memory.

## Flynn's Taxonomy

Four processing modes:

 1. Single instruction stream, single data stream (SISD)
 2. Single instruction stream, multiple data stream (SIMD)
 3. Multiple instruction stream, single data stream (MISD)
 4. Multiple instruction stream, multiple data stream (MIMD)

# Terminology and Basic Algorithms - Chapter 5

Classifications of distributed application execution:

 * Centralized algorithm

    Typicallly a client/server algorithm or an algorithm where one or
    more nodes does a predominant amount of work.

 * Distributed algorithm

   Each processor plays and equal rol ein message, time, and space overhead.

 * Symmetric algorithm

   All processors in a symmetric algorithm executve the same logical functions.

 * Asymmetric algorithm

   Each processor execute logically different, but perhaps overlapping, functions.

 * Anonymous alogrithm

   Processes nor processors use their process identifiers to make any execution
   decision.  Hard to design where a leader needs to be elected.


 * Uniform algorithm

   Does not use n, the number of processes in a system, as parameter in its code.
   Allows scalability transparency and processes can join or leave without intruding
   on other processes.

 * Adapative algorithm

   When an algorithm complexity can be described as a subset of nodes (k, where
   k <= n), then the algorithm is adaptive.

*Deterministic* versus *non-deterministic*.  Deterministic specifies the source from
which it wants to receive a message.  Non-deterministic receive primitive can receive
a message from any source.  A distributed program that contains no non-deterministic
receves has a deterministic execution, otherwise if it contains at least one it is
said to have a non-deterministic execution.

Execution inhibition (aka freezing).

*Synchronous* versus *asynchronous*.  A synchronous satisfies:

  * There is a known upper bound on the message communication **delay**.
  * There is a known bounded **drift rate for the local clock** of each
    processor.
  * Ther eis a known upper bound on the **process execution time**.

Complexity measures:

  * **Space complexity**: memory requirement per node.
  * **Systemwide space complexity**:  not space n * corresponding space
    complexity.
 
## Sync 1-initiator ST (flooding)

**Overview:** A process that proceeds in rounds.  Assumes a designated root node
that initiates the algorithm with a QUERY flood messages to identify tree edges.
A parent node is that node from which a QUERY is first received.  If multiple
QUERY messages are received in a round, one sender is chosen at random.  It
terminates after all rounds are executed.  The algorithm could be modified
so that a process exits once it sets its parent variable.

**Complexity:**
 * Local space complexity at a node is on the order of the degree of edge
   incidence.
 * Local time complexity at a node is on the order of diamater + degree of
   edge incident
 * Global space complexity is the sum of local space complexities.
 * Algorithm sends at least one message per edge, and at most, two messages
   per edge.  Thus the number of messages is between *l* and *2l*.
 * Message time compexity is *d* rounds or message hops.

**Algorithm:**

```
# local variables
int (visited, depth) = 0
int parent           #  undefined (null)
set of int Neighbors = (set of neighbors)

# message types
QUERY

if i == root then
    visited = 1
    depth = 0
    send QUERY to Neighbors

for round = 1 to diameter do
    if visited == 0 then
        if any QUERY messages arrive then
            parent  = random(node from which a QUERY was received)
            visited = 1
            depth = round

            # send QUERY to Neighbors we didn't just receive a QUERY from
            send QUERY to Neighbors \{senders of QUERYs received in this round}

    delete any QUERY messages that arrived in this round
```
 
## Async 1-initiator ST (flooding)

**Overview:** Assumes a designated root node which initiates the algorithm
by sending QUERY messages to neighbors.  A parent node is a node from which
a QUERY message first arrives.  ACCEPT messages are sent to parent QUERY
messages.  All other QUERY messages receive a REJECT reply.  A node terminates
when all non-parent neighbors have responded to a QUERY message.  There is no
concept of a round since this is an asynchronous sytem.

**Complexity:**

 * Node Local space complexity is of the order of the degree of edge incidence.
 * Node local time complexity is of te order of the degree of edge incidence.
 * Global space complexity is the sum of local space complexities.
 * At least two (QUERY+response) messages per link, at most four.  Total number
   of messages is between *2l* and *4l*.
 * In async we cannot make any claims about bounded time complexity.
 
**Algorithm:**

```
# local variables
int parent   # undefined (null)
set of int Children,Unrelated = {}  # empty
set of int Neighbors = (set of Neighbors)

# message types
QUERY, ACCEPT, REJECT

# when predesignated root node wants to initiate algorithm
if (i == root and parent == null)
    send QUERY to all neighbors
    parent = i

# when QUERY arrives from j
if parent == null
    parent = j
    sent ACCEPT to j
    send QUERY to all neighbors except j
    if (Children U Unrelated) = (Neighbors\{parent})
        terminate
else
    send REJECT to j

# when ACCEPT arrives from j
Children = Children U {j}
if (Children U Unrelated) == (Neighbors\{parent}})
    terminate

# when REJECT arrives from j
Unrelated = Unrelated U {j}
if (Children U Unrelated) == (Neighbors\{parent})
    terminate
```

## Async concurrent-initiator ST (flooding)

**Overview:**We modify the Async 1-initiator ST algorithm to allow
for concurrent, independent initiators.  A primary concern is handle
multiple STs forming and either merging them or finding a way for a
single tree to emerge by having one tree take priority.  Since merging
is complex we opt for having one ST emerge through the use of comparing
unique root ids for tree selection.  Only the root knows when the
algorithm can be terminated.  The root can send a special message along
the constructed tree to inform nodes.

**Complexity:**

 * Time complexity is *O(l)* messages and the number of messgaes is
   *O(nl)*.

**Algorithm:**

```
# local variables
int parent,myroot                   # undefined (null)
set of int Children,Unrelated = {}  # empty
set of int Neighbors = (set of Neighbors)

# message types
QUERY, ACCEPT, REJECT

# when node wants to initiate the algorithm as root
if parent == null
    send QUERY(i) to all neighbors
    parent,myroot = i

# when QUERY(newroot) arrives from j
if myroot < newroot
    parent = j
    myroot = newroot
    Children,Unrelated = 0
    send QUERY(newroot) to all neighbors except j
    if Neighbors == {j}
        send ACCEPT(myroot) to j
        terminate  # leaf node
else
    send REJECT(newroot) to j

# when ACCEPT(newroot) arrives from j
if newroot == myroot
    Children = Children U {j}
    if (Children U Unrelated) == (Neighbors\{parent})
        if i == myroot
            terminate
        else
            send ACCEPT(myroot) to parent

# when REJECT(newroot) arrives from j
if newroot == myroot
    Unrelated = Unrelated U {j}
    if (Children U Unrelated) == (Neighbors\{parent})
        if i == myroot
            terminate
        else
            send ACCEPT(myroot) to parent
```

## Async DFS ST

**Overview:** This algorithm is similar to the previous async concurrent
initiator ST algorithm, except it uses a depth first search (DFS) to
build the spanning tree.  Terminator is as the previous algorithm, where
the root only has definitive knowledge.

**Complexity:**

 * Time complexity is *O(l)* messgaes and the number of messages is
   *O(nl)*.

**Algorithm:**

```
# local variables
int parent,myroot        # undefined (null)
set of int Children = {} #empty
set of int Neighbors,Unknown = set of neighbors

# message types
QUERY, ACCEPT, REJECT

# when a node initiates the algorithm as root
if parent == null
    send QUERY(i) to i   # send to itself

# when QUERY(newroot) arrives from j
if myroot < newroot
    parent = j
    myroot = newroot
    Unknown = set of neighbors
    Unknown = Unknown\{j}
    if Unknown != 0
        delete some x from Unknown
        send QUERY(myroot) to x
    else
        send ACCEPT(myroot) to j
else if myroot == newroot
    send REJECT to j

# when ACCEPT(newroot) or REJECT(newroot) arrives from j
if newroot == myroot
    if ACCEPT message arrived
        Children = Children U {j}
    if Unknown == 0
        if parent != i
            send ACCEPT(myroot) to parent
        else
            set i as the root
            terminate
    else
        delete some x from Unknown
        send QUERY(myroot) to x
```

## Broadcast & convergecast on tree

A spanning tree is useful for distributing (via broadcast) and
collecting (via convergecast) information to/from all nodes.
Convergecast can be used to discover some min/max value at each
node for instance.  It might also be used for leader election.

A broadcast is specified as follows:

 * Root sends information to be broadcast to all children.  Terminate.
 * When a (nonroot) node receives information from its parent, copy
   it and forward it to its children.  Terminate.

Convergecast is usually initiated by a root broadcast request.  A
coveragecast alogirhm is specified as follows:

 * Leaf node sends its report to its parent.  Terminate.
 * At a nonleaf node that is not the root, when a report is received
   from all child nodes, the collective report is sent to the parent.
   Terminate.
 * At the root, when a report is received from all child nodes, the
   gloal function is evaluated using the reports.  Terminate.

## Sync 1-source shortest path: synchronous Bellman-Ford

**Overview:** Given a weighted graph of unidirectional links, the
Bellman-Ford algorithm finds the shortest path from a given node to all
other nodes.  The algorithm is correct when there are no cyclic paths
having negative weight.  The assumed toplogy is (*N*, *L*) is not
known to any process.  A node only knows about the incident links and
their weights.  It is assumed that the processes know the number of odes
|*N*| = *n*.  The algorithm is uniform and the assumption on *n*
is required for termination.

**Complexity**: time complexity is *n - 1* rounds, message complexity
is *(n - 1)l* messages.

**Algorithm**:


```
# local variables
int length = infinity
int parent = parent
set of int Neighbors = set of neighbors
set of int { weight_i,j, weight_j,i | j E Neighbors } = known values of incident links

# message types
UPDATE

if i = i_0
    length = 0

# received UPDATE ( i_0, length_j) from j 
for round = 1 to n - 1
    send UPDATE(i, length) to all neighbors
    await UPDATE(j, length_j) from each j E Neighbors
    for each j E Neighbors
        if length > (length_j + weight_j,i)
            length = length_j + weight_j,i
            parent = j
```

## Distance Vector Routing

When network graph is dynamically changing, the graph never stabilizes.
Maintain an array of LENGTH[1..n] where LENGTH[k] denotes the length
measure from k.  Maintain similar array for PARENT.  Processes exchange
distance vectors periodically since this is async.  Computes shortest
path to itself.

## Async 1-source shortest path: asynchronous Bellman-Ford

**Complexity:** O(*n^^3*)

```
# local variables
int length = infinity
set of int Neighbors = set of neighbors
set of int [weight_i,j, weight_j,i] | j E Neighbors] = known values of weights of incident links

# message types
UPDATE

if i == i_0
    length = 0
    send UPDATE(i_0, 0) to all neighbors
    terminate

# when UPDATE(i_0, length_j) arrives from j
if (length > (length_j + weight_j,i)
    length = length_j + weight_j,i
    parent = j
    send UPDATE(i_0, length) to all neighbors
````

## All sources shortest path: Floyd-Warshall

Computers all pairs shortest-path with length vector.

**Algorithm:**

```
for pivot == 1 to n
    for s == 1 to n
        for t == 1 to n
            if LENGTH[s,pivot] + LENGTH[pivot,t] < LENGTH[s,t]
                LENGTH[s,t] = LENGTH[s,pivot] + LENGTH[pivot,t]
            VIA[s,t] = VIA[s,pivot]
```

## Toueg's asynchronous distributed Floyd-Warshall all-pairs shortest paths

```
# local variables
int LEN[1..n]
int PARENT[1..n]
set of int Neighbors = set of neighbors
int pivot,nbh = 0

# message types
IN_TREE(pivot), NOT_IN_TREE(pivot), PIV_LEN(pivot, PIVOT_ROW[1..n])

for pivot == 1 to n
    for each neighbor nbh E Neighbors
        if PARENT[pivot] == nbh
            send IN_TREE[pivot] to nbh
        else
            send NOT_IN_TREE(pivot) to nbh
    await IN_TREE or NOT_IN_TREE message from each neighbor
    if LEN[pivot] != infinity
        if pivot != i
            receive PIV_LEN(pivot, PIVOT_ROW[1..n]) from PARENT[pivot]
        for each neighbor nbh E Neighbors
            if IN_TREE message was received from nbh
                if pivot == i
                    send PIV_LEN(pivot, LEN[1..n]) to nbh
                else
                    send PIV_LEN(pivot, PIVOT_ROW[1..n]) to nbh
        for t = 1 to n
            if LEN[pivot] + PIVOT_ROW[t] < LEN[t]
                LEN[t] = LEN[pivot] + PIVOT_ROW[t]
                PARENT[t] = PARENT[pivot
```

## Sync, async constrained flooding

**Overview:** This is essentially the link-state protocol.  We send a
sequence number in the updates, which is kept track of.  Older ones are
ignored, otherwise it is flooded out all links.


**Complexity:* Message complexity is *2l* messages in the worst case
where each message *M* has overhead *O*(1).  Time complexity is diameter
*d* of sequential hops.

Asynchronous flooding:


```
# local variables
int SEQNO[1..n] = 0
set of int Neighbors = set of neighbors

# message types
UPDATE

# to send a message M
if i == root
    SEQNO[i] = SEQNO[i] + 1
    send UPDATE(M, i, SEQNO[i]) to each j E Neighbors

# when UPDATE(M, j, seqno_j) arrives from k
if SEQNO[j] < seqno_j
    # process the message M
    SEQNO[j] = seqno_j
    send UPDATE(M, j, seqno_j) to Neighbors\{k}
else
  # discard the message
```

Synchronous flooding:

```
# local variables
int STATEVEC[1..n] = 0
set of int Neighbors = set of neighbors

# message types
UPDATE

STATEVEC[i] = local value
for round = 1 to diameter d
    send UPDATE(STATEVEC[1..n]) to each j E Neighbors
    for count = 1 to |Neighbors|
        await UPDATE(SV[1..n]) from some j E Neighbors
        STATEVEC[1..n] = max(STATEVEC[1..n], SV[1..n])
```

## MST, sync

Minimum-weight spanning tree algorithm, synchronous.  Minimizes the cost
of transmission from any node to any node.  Assumes entire graph is
available for examination.  A *forest* (i.e. a disjoint union of trees) is
a graph in which any pair of nodes is connectd by at most one path.  A
*spanning forest* of an undirected graph *(N, L)* is a maximal forest of
*(N, L)*.  When a spanning forest is connected, it becomes a spanning
tree.

## MST, async

## Synchronizers: simple, alpha, beta, gamma

Generic class of algorithms that transform algorithms from synchronous
to asynchronous are known as synchronizers.

Observations:

 * Consider only failure-free systems.
 * They can be complex.  Tailor made async algo may be more efficient.

**Simple**: each process sends one and only one message to every
neighbor in a round.

**alpha:** every message requires an acknowledgement.

**beta**: assumes a rooted spanning tree

**gamma: combines alpha and beta.

Process safety: process *i* is safe in a round if all messages sent by
*i* have been received.  Requires acknowledges.

## Maximal independent set (MIS) - Luby's Algorithm

A maximal set of nodes in a graph (can't add other nodes since they would
have an incident edge to an existing node).

## Connected dominating set (CDS)

A set in which every node has a vertex to one of the nodes in the set.
Finding a set of size k <|N| is NP complete.

## Compact routing tables

Routing tables could be created such that there is a route for each
destination n.  Scaling this, but reducing the size of the routing
table can be done by several schemes.

**Hiearchical routing:** organize graph is clusters, where a clusterhead
represents the cluster at the next higher level.

**Tree-labeling:** logical tree topology.

**Interval routing:** eliminate need to send packets only over tree
edges.

**Prefix routing:** overcomes drawback of interval routing (not CIDR,
but like it).

## Leader election: LCR algorithm

Often needed because algorithms are not completely symmetric.  Often
uses a ring topology.

LCR algorihtm:

```
# variables
boolean partiicpate = false

# message types
PROBE integer
SELECTED integer

# when a process wakes up to participate in leader election
send PROBE(i) to right neighbor
participate = true

# when a PROBE(k) message arrives from left neighbor Pj
if participate = false
    execute step 1 (wake up, send to right neighbor) first
if i > k
    discard the probe
else if i < k
    forward PROBE(k) to right neighbor
else if i == k
    declare i is the leader
    circulate SELECTED(i) to right neighbor

# when a SELECTED(x) message arrives from left neighbor
if x != i
    note x as leader and forwadr message to right neighbor
else
    do not forward the SELECTED message
```

## Challenges in designing distributed graph algorithms

Graph changes and failures complicate matters.

### Object replication

On tree overlay, expand tree when read cost greater than write cost.
Shrink tree when write cost is higher.


# Global State and Snapshot Recording Algorithms - Chapter 4

A consistent global state or consistent snapshot is used to analyze,
test, and verify properties associated with distributed executions.
Without a globally shared memory and clock this is a non-trivial
challenge.

This chapter discusses the issues in consistent state (or snapshots)
and algorithms used to obtain a consistent state or snapshot.

## Chandy-Lamport algorithm

```
# marker sending rule for process p_i

1. Process p_i records its state
2. for each outgoing channel C on which a marker has not been sent, p_i
   sends a marker along C before p_i sends further messages along C.

# market receiving rule for process p_j

On receiving a marker along channel C:
  if p_j has not record its state
      Record the state of C as the empty set
      Execute the "marker sending rule"
  else
      Record the state of C as the set of messages
      received along C after p_j,s state was recorded
      and before p_j received the marker along C
```

## ~~Spezialetti-Kearns algorithm~~
## ~~Lai-Yang algorithm~~
## ~~Mattern's algorithm~~
## ~~Acharya-Badrinath algorithm~~
## ~~Alagar-Venkatesan algorithm~~
## ~~Manivannan-Netzer-Singhal algorithm~~
## ~~R-graph~~~~~~~~

# Message Ordering and Group Communication - Chapter 6

## Asynchronous and FIFO Executions
## Casual Order
## Synchronous Executions
## Asynchronous Execution with Synchronous Communications
## RSC Executions
## Crown
## ~~Bagrodia's Algorithm for Binary Rendezvous~~
## Raynal-Schiper-Toueg (RST) Algorithm
## Optimal KS Algorithm for CO
## Total Message Order
## Multicast
## Reverse Path Forwarding (RPF)
## Steiner Trees

# Consensus and Agreement
## Consensus Algorithm for Crash Failures (MP, synchronous)
## Upper Bound on Byzantine Processes
## Byzantine Generals (recursive formulation), (sync, msg-passing)
## Byzantine Generals (iterative formulation), Sync, Msg-passing
## The Phase King Algorithm
## Terminating Reliable Broadcast (TRB)
## Epsilon Consensus (msg-passing, async)
## Asynchronous Renaming
## Reliable Broadcast
## Shared Memory Consensus (async)
## Wait-free SM Consensus using Shared Objects
## Two-process Wait-free Consensus using FIFO Queue
## Wait-free Consensus using Compare & Swap
## Read-Modify-Write (MRW) Abstraction
## Nonblocking Universal Algorithm
## Wait-free Universal Algorithm
## Async Wait-free Renaming using Atomic Shared Object
## The Spitter

# Reasoning with Knowledge
## Muddy Children Puzzle
## Kripke Structures
## Common Knowledge for Asynchronous Systems
## Concurrent Knowledge
### Snapshot-based Algorithm
### Three-phase Send Inhibitory Algorithm
### Three-phase Send Inhibitory Tree Algorithm
## Inhibitory Ring Algorithm
## Knowledge Transfer
## Knowledge and Clocks
### Scalar clocks
### Vector clocks
### Matrix Clocks

# Peer-to-peer computing and overlay graphs
## Data Indexing and Overlays
### Centralized Indexing
Napster was an early example.
### Distributed Indexing
Distributed hash table (DHT) proposals popular here.
#### Structured Overlays
#### Unstructued Overlays
### Local Indexing
Gnutella uses local indexing.
## Chord Distributed Hash Table
## Content Addressable Networks (CAN)
## Tapestry
