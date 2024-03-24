.. include:: substitutions.rst

Introduction
============


Leader election is a procedure for picking a unique leader process among multiple processes so that the
leader knows that it has been picked as the leader and the other processes knows that they are not the leader.

The leader election problem is one of the fundamental problems in the field of distributed systems, helping us to
achieve coordination and synchronization among multiple processes.
For example, in a distributed database system, a leader process can be responsible for coordinating transactions and ensuring consistency among the database replicas.
The reason the leader election problem is important is that it helps to avoid conflicts and inconsistencies among the processes in a distributed system.

The challenging part of the leader election problem is to ensure that only one process is picked as the leader at a time,
even in the presence of failures and network partitions. Additionally, the parameters for a network can vary greatly, creating the
need for many case specific algorithms to solve the leader election problem efficiently.

The algorithm we explain and implement, the Itai-Rodeh Leader Election Algorithm [Itai1981]_, is such an algorithm to pick a leader process among multiple processes in a distributed system.
Itai-Rodeh Leader Election Algorithm aims to solve the leader election problem on asynchronous, anonymous, unidirectional rings consisting of n>2 processes where the length of the ring is known to each process beforehand.
The configuration we consider is:

* The processes are anonymous, meaning that they do not have unique identifiers given to them.
* The communication channel is asynchronous, meaning that there is no bound on the message delay. Also, the Itai-Rodeh Leader Election Algorithm does not assume any constraint about the ordering of the messages in the communication channels, e.g we cannot assume the channels are FIFO.
It would be beneficial to note that the original paper [Itai1981]_ does provide prooves for both synchronous and asynchronous channels, but we'll implement asynchronous version.
* The topology of the network is a ring. In such a topology, all processes are only connected to a single one of their neighbors.
* The communication channel is unidirectional, meaning that the processes can only send messages to their neighbors in the ring.




.. [Fokkink2005]  Fokkink, W., & Pang, J. (2005). Simplifying Itai-Rodeh Leader Election for Anonymous Rings. Electronic Notes in Theoretical Computer Science, 128(6), 53–68. doi:10.1016/j.entcs.2005.04.004
.. [Itai1981] A. Itai and M. Rodeh. Symmetry breaking in distributive networks. In Proc. FOCS’81, pp. 150–158. IEEE Computer Society, 1981.

