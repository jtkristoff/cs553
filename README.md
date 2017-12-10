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

## Sync 1-source shortest path

## Distance Vector Routing
## Async 1-source shortest path
## All sources shortest path: Floyd-Warshall
## Sync, async constrained flooding
## MST, sync
## MST, async
## Synchronizers: simple, alpha, beta, gamma
## Maximal independent set (MIS)
## Connected dominating set (CDS)
## Compact routing tables
## Leader election: LCR algorithm
## Challenges in designing distributed graph algorithms
### Object replication

# Global State and Snapshot Recording Algorithms (Chapter 4)

A consistent global state or consistent snapshot is used to analyze,
test, and verify properties associated with distributed executions.
Without a globally shared memory and clock this is a non-trivial
challenge.

This chapter discusses the issues in consistent state (or snapshots)
and algorithms used to obtain a consistent state or snapshot.

## Chandy-Lamport algorithm


## ~~Spezialetti-Kearns algorithm~~
## ~~Lai-Yang algorithm~~
## ~~Mattern's algorithm~~
## ~~Acharya-Badrinath algorithm~~
## ~~Alagar-Venkatesan algorithm~~
## ~~Manivannan-Netzer-Singhal algorithm~~
## ~~R-graph~~~~~~~~

# Message Ordering and Group Communication
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
