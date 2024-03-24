.. include:: substitutions.rst

|itai_rodeh_election|
=========================================



Background and Related Work
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Famously, Angluin et al.  shown that no algorithm can result in a leader in an arbitrary sized ring
of anonymous processes within a finitely bounded time [Angluin1980]_.
The result is interpreted as the symmetry in such a system can be broken either by unbounded computational time or
breaking the correctness of the algorithm. Following the result, some probabilistic algorithms have been proposed such as the
the paper we discuss and implement in here, the Itai-Rodeh election algorithm [Itai1981]_, an improved version of the original Itai-Rodeh election algorithm
with the additional assumption of FIFO communication channels [Fokkink2005]_, and [Abrahamson1986]_. TODO add more related work.

.. [Angluin1980] ANGLUIN, D. (1980), Local and global properties in networks of processes, in “12th Annual ACM Sympos. on Theory of Computing, Los Angeles, California,” pp. 82-93.
.. [Abrahamson1986] Abrahamson, K., Adler, A., Higham, L., & Kirkpatrick, D. (1986). Probabilistic solitude verification on a ring. Proceedings of the Fifth Annual ACM Symposium on Principles of Distributed Computing, 161–173. Presented at the Calgary, Alberta, Canada. doi:10.1145/10590.10604


Distributed Algorithm: |itai_rodeh_election|
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Itai-Rodeh Leader Election Algorithm is presented in :ref:`Algorithm <ItaiRodehLabel>`.

Itai-Rodeh Leader Election Algorithm requires each process to maintain the following parameters:

* :math:`id_i \in {1,...,k}`, for some :math:`k \geq 2`, is its identity
* :math:`state_i \in` {active, passive, leader} is its current state
* :math:`round_i \in N` represents the number of the current election round.

At the start of each election round, each process is active and sends their identities randomly as :math:`id_i \in {1,...,k}`, for some :math:`k \geq 2`.
After that, each process sends the following message to its communication channel:
(id,round,hop,bit) where

* id is the process's identity
* round is the current round number
* hop is the number of hops the message has traveled
* bit is flag that is set to 1 initially, but is set to 0 when the process receives a message
with a same id that it has to prevent two processes to elected as the leader at the same time.

.. _ItaiRodehPseudocodeLabel:

Upon receiving a message, the process acts as follows:

* If it is passive, it passes the message by only increasing the hop count by one.

* If it is active, it checks:

   * If the hop count == N, and bit == 1, it becomes leader.

   * If the hop count == N, and bit == 0, process infers that at least one other process picked the same biggest number for their id,
   so no leader could have been elected in this round. It starts a new round

   * If round and id is the same as the received message while hop < N, it emits the message (id, round, hop+1, 0) to the channel.

   * If round in the message is bigger than the process knows, process becomes passive and passes the message by increasing the hop count.

   * If the round in the message is equal to the state of the process, but the id of the process is smaller than the id in the message, process becomes passive and passes the message by increasing the hop count.

   * In all other cases, the process purges the message it received.


.. _ItaiRodehLabel:

.. code-block:: RST
    :linenos:
    :caption: Itai-Rodeh Leader Election Algorithm [Itai1981]_.

    TODO Edit following with the implementation

    Implements: Itai-Rodeh Leader Election Algorithm : cf
    Uses: ?LinkLayerBroadcast Instance: lbc
    Events: ?Init, MessageFromTop, MessageFromBottom
    Needs: ?

    OnInit: () do
    
    OnMessageFromBottom: ( m ) do
        Trigger lbc.Broadcast ( m )
    
    OnMessageFromTop: ( m ) do
        Trigger lbc.Broadcast ( m )



Example
~~~~~~~~

Provide an example for the distributed algorithm.

Correctness
~~~~~~~~~~~

Let's study algorithm in terms of its Correctness, safety, liveness and fairness properties.

**Correctness** is the property of the algorithm that ensures that the algorithm always produces the correct result.
As it has been discussed, Itai-Rodeh Leader Election Algorithm is a Las Vegas algorithm that terminates with probability 1,
and always produces correct result.

For the algorithm to be incorrect, a termination configuration either ends with no leader, or multiple leaders.

* For the **case of no leader**, following the pseudocode shared in the :ref:`Pseudo-code <ItaiRodehPseudocodeLabel>`,
  each process needs to become passive. To become passive, a process needs to receive either a message from a bigger round, or a message with a bigger id.
  Since the message containing the bigger id is sent from another process, this process cannot become passive, so there should at least a process that does not
  become passive.

* For the **case of multiple leaders**, two or more processes should get the same biggest id in a round, but in this case
  since the hop counter will be less than the size of the ring, the uniqueness bit we use will be emitted as false and
  no process will be elected as the leader.

**Safety** is the property of the algorithm that ensures that the algorithm never produces an incorrect result.
In the case of Itai-Rodeh Leader Election Algorithm, the algorithm itself does not describe any information
about the formation of the ring and its recovery in case of a node failure. Assuming that there can be no
partitioning in the ring, the safety of the algorithm depends on handling of the link and process failures.

For process/link failures:
   * If a process/link becomes unavailable, algorithm cannot proceed thus cannot produce any result.
   * If the process/link produces incorrect results(e.g it does not alter hop counter,
     or does not flip the uniqueness bit) the result produced by the algorithm would become incorrect.
     (TODO Author here does not sure if byzantine fault tolerance is required for the algorithm to be safe, or if it is enough to handle the process failures.)

Following the failure scenarios, we can conclude that the algorithm is safe, as it does not produce incorrect results.

**Liveness** is the property of the algorithm that ensures that the algorithm always terminates with expected results.

As it has been discussed in the safety section, the Itai-Rodeh Leader Election Algorithm does not produce any results
when a failure happens on a process or a link. This characteristic makes the algorithm not always live.

**Fairness** is the property of the algorithm that ensures that the algorithm treats all processes fairly. In our case,
each of the processes randomly picks an id for them in each round from the same set of ids, so the algorithm is fair.


Complexity 
~~~~~~~~~~

Present theoretic complexity results in terms of number of messages and computational complexity.




.. admonition:: EXAMPLE 

    Snapshot algorithms are fundamental tools in distributed systems, enabling the capture of consistent global states during system execution. These snapshots provide insights into the system's behavior, facilitating various tasks such as debugging, recovery from failures, and monitoring for properties like deadlock or termination. In this section, we delve into snapshot algorithms, focusing on two prominent ones: the Chandy-Lamport algorithm and the Lai-Yang algorithm. We will present the principles behind these algorithms, their implementation details, and compare their strengths and weaknesses.

    **Chandy-Lamport Snapshot Algorithm:**

    The Chandy-Lamport :ref:`Algorithm <ChandyLamportSnapshotAlgorithm>` [Lamport1985]_ , proposed by Leslie Lamport and K. Mani Chandy, aims to capture a consistent global state of a distributed system without halting its execution. It operates by injecting markers into the communication channels between processes, which propagate throughout the system, collecting local states as they traverse. Upon reaching all processes, these markers signify the completion of a global snapshot. This algorithm requires FIFO channels. There are no failures and all messages arrive intact and only once. Any process may initiate the snapshot algorithm. The snapshot algorithm does not interfere with the normal execution of the processes. Each process in the system records its local state and the state of its incoming channels.

    1. **Marker Propagation:** When a process initiates a snapshot, it sends markers along its outgoing communication channels.
    2. **Recording Local States:** Each process records its local state upon receiving a marker and continues forwarding it.
    3. **Snapshot Construction:** When a process receives markers from all incoming channels, it captures its local state along with the incoming messages as a part of the global snapshot.
    4. **Termination Detection:** The algorithm ensures that all markers have traversed the system, indicating the completion of the snapshot.


    .. _ChandyLamportSnapshotAlgorithm:

    .. code-block:: RST
        :linenos:
        :caption: Chandy-Lamport Snapshot Algorithm [Fokking2013]_.
                
        bool recordedp, markerp[c] for all incoming channels c of p; 
        mess-queue statep[c] for all incoming channels c of p;

        If p wants to initiate a snapshot 
            perform procedure TakeSnapshotp;

        If p receives a basic message m through an incoming channel c0
        if recordedp = true and markerp[c0] = false then 
            statep[c0] ← append(statep[c0],m);
        end if

        If p receives ⟨marker⟩ through an incoming channel c0
            perform procedure TakeSnapshotp;
            markerp[c0] ← true;
            if markerp[c] = true for all incoming channels c of p then
                terminate; 
            end if

        Procedure TakeSnapshotp
        if recordedp = false then
            recordedp ← true;
            send ⟨marker⟩ into each outgoing channel of p; 
            take a local snapshot of the state of p;
        end if


    **Example**

    DRAW FIGURES REPRESENTING THE EXAMPLE AND EXPLAIN USING THE FIGURE. Imagine a distributed system with three processes, labeled Process A, Process B, and Process C, connected by communication channels. When Process A initiates a snapshot, it sends a marker along its outgoing channel. Upon receiving the marker, Process B marks its local state and forwards the marker to Process C. Similarly, Process C marks its state upon receiving the marker. As the marker propagates back through the channels, each process records the messages it sends or receives after marking its state. Finally, once the marker returns to Process A, it collects the markers and recorded states from all processes to construct a consistent global snapshot of the distributed system. This example demonstrates how the Chandy-Lamport algorithm captures a snapshot without halting the system's execution, facilitating analysis and debugging in distributed environments.


    **Correctness:**
    
    *Termination (liveness)*: As each process initiates a snapshot and sends at most one marker message, the snapshot algorithm activity terminates within a finite timeframe. If process p has taken a snapshot by this point, and q is a neighbor of p, then q has also taken a snapshot. This is because the marker message sent by p has been received by q, prompting q to take a snapshot if it hadn't already done so. Since at least one process initiated the algorithm, at least one process has taken a snapshot; moreover, the network's connectivity ensures that all processes have taken a snapshot [Tel2001]_.

    *Correctness*: We need to demonstrate that the resulting snapshot is feasible, meaning that each post-shot (basic) message is received during a post-shot event. Consider a post-shot message, denoted as m, sent from process p to process q. Before transmitting m, process p captured a local snapshot and dispatched a marker message to all its neighbors, including q. As the channels are FIFO (first-in-first-out), q received this marker message before receiving m. As per the algorithm's protocol, q took its snapshot upon receiving this marker message or earlier. Consequently, the receipt of m by q constitutes a post-shot event [Tel2001]_.

    **Complexity:**

    1. **Time Complexity**  The Chandy-Lamport :ref:`Algorithm <ChandyLamportSnapshotAlgorithm>` takes at most O(D) time units to complete where D is ...
    2. **Message Complexity:** The Chandy-Lamport :ref:`Algorithm <ChandyLamportSnapshotAlgorithm>` requires 2|E| control messages.


    **Lai-Yang Snapshot Algorithm:**

    The Lai-Yang algorithm also captures a consistent global snapshot of a distributed system. Lai and Yang proposed a modification of Chandy-Lamport's algorithm for distributed snapshot on a network of processes where the channels need not be FIFO. ALGORTHM, FURTHER DETAILS

.. [Fokking2013] Wan Fokkink, Distributed Algorithms An Intuitive Approach, The MIT Press Cambridge, Massachusetts London, England, 2013
.. [Tel2001] Gerard Tel, Introduction to Distributed Algorithms, CAMBRIDGE UNIVERSITY PRESS, 2001
.. [Lamport1985] Leslie Lamport, K. Mani Chandy: Distributed Snapshots: Determining Global States of a Distributed System. In: ACM Transactions on Computer Systems 3. Nr. 1, Februar 1985.