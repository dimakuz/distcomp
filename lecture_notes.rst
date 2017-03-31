Distributed Algorithms
======================

:Author: Dima Kuznetsov

Lecture 1
~~~~~~~~~

Intro
-----

Key differences between centralized and distributed computation:

* Asynchronicity of the network.
* Fault / unavailability of nodes.

This adds uncertainty to distributed algorithms.

Models of distributed computation
---------------------------------

We can come up with many different models to describe a distributed computation
and its environment, e.g.

How nodes share information:

* **Message passing** - each node is connected to a group of other nodes, each
  via a different port, and can send/receive messages from those ports.
* **Shared memory** - all nodes have access to a piece of shared memory, and
  can exchange information by writing and reading from the memory.

What knowledge the nodes have:

* Does each one have a unique ID?
* Do they know the identity of other computers?
* Do they know the network topology?
* Do they know the total number of nodes?

Do nodes suffer faults:

* **No faults** - all nodes are always operational and run their computation
  as programmed.
* **Fail/Stop** - a node can occaionally fail. The failure is complete, i.e. a
  failed node goes off-line.
* **Byzantine** - nodes can fail but remain on-line, yielding malicious or
  invalid results.
* **Rational agents** - nodes can "cheat" in order to affect the result of an
  algorithm, e.g. a node affects leader election to minimize the chance of
  being chosen leader to do less computation, or vice versa, maximizing the
  change to be in control of task distribution.

We can also talk about complexity of distributed algorithms:

* *Time complexity* - How long does it take the algorithm to finish?
* *Message complexity* - How many messages are sent in the running time of the
  algorithm.


Message passing
---------------

In the first part of this course we'll focus on problems modeled with message
passing, those include:

* Graph coloring
* DNS
* Shortest path

Each node is modeled as a graph vertex and each port between 2 nodes as graph
edge:

::

 +------+
 |Node 1|
 |      +--------+
 +--+---+        |          +---------+
    |            +----------+ Node 2  |
    |                       |         |
    |                       +----+----+
    |                            |
    |                            |
    |            +---------------+
    |            |
    |            |
    |       +----+--+
    +-------+ Node 3|
            |       +-----------+          +--------+
            +-------+           |          | Node 4 |
                                +----------+        |
                                           +--------+

Each node is composed of the following:

* A *computation unit*, i.e. a CPU / something that runs our program.
* An *incoming message queue* where all incoming messages arrive, it also
  saves where each message came from.
* Bidirectional *ports* that connect the node to its neighbors.

::

 +---------------------------------+
 | Node 1                          | Port 1
 |                                 +---------------->
 |        +------------+           <----------------+
 |        |            |           |
 |        |    CPU     |           |
 |        |            |           | Port 2
 |        |            |           +---------------->
 |        +------------+           <----------------+
 |                                 |
 |                                 |
 |   FIFO                          | Port 3
 |  +---+---+---+---+---+-------+  +---------------->
 |  |   |   |   |   |   |          <----------------+
 |  |   |   |   |   |   |          |
 |  +---------------------------+  |
 +---------------------------------+


The model itself is event driven, a node handles all incoming messages in the
order they arrive. Each message is handled by a piece of code / program:

::

  upon receiving message M on port P:
      do something

The program can alter node's state (variables), do computations, and send
messages to adjacent nodes.

Some of the nodes are designated as *initiators* - nodes that start the
computation by sending the initail messages.

We'll assume that all events in the network are ordered, and nothing is really
simultaneous.

Broadcast and spanning tree example
-----------------------------------

We'll describe an aglorimth that performs a broadcast, that is sending a\
message to all of the nodes in the network.

Initiator node code:

::

    initailization(M):
        ackCounter = 0
        send M to all neighbors

    upon receiving Ack on port j:
        ackCounter++
        if ackCounter == | neigh |:
            Terminate()

On other nodes:

::

    initializtion():
        ackCounter = 0
        parent = nil

    upon receiving M != Ack on port j:
        if parent is nil:
            parent = j
            if | neighs | == 1:
                send Ack to j
            else:
                send M to all neighbors except j
        else:
            send Ack to j

    upon receiving Ack on port j:
        ackCounter++
        if ackCounter == | neigh | - 1:
            send Ack to parent


By using parent on each of the nodes we get a spanning tree (all nodes except
root have a single parent, exactly N - 1 edges).

Message M traves twice on all edges except for the tree edges, where it
traverses once.

Total message complexity is :math:`2|E| - (|V| - 1)`

