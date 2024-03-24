.. include:: substitutions.rst
========
Abstract
========


Leader election problem is a fundamental problem of distributed computing where you want to pick a process as a leader among others.
In many cases, the decisions or transactions made by the leader ensures the consistency of the system by preventing conflicts.
In this project, we implement the Itai-Rodeh Leader Election algorithm in Python. The algorithm is a Las Vegas algorithm that
works on asynchronous, anonymous, unidirectional ring consisting of n>2 processes and when the length of the ring is known.
The basic idea of the algorithm is that each process picks a random id for themselves from a predefined range of values
and shares that information with the ring with some additional properties like the round number it thinks it is in,
hop counter to check the originators of the messages and a flag to check if it saw any conflicting ids. The Itai-Rodeh leader
election algorithm solves the leader election problem in the mentioned setup with a probability of 1.

