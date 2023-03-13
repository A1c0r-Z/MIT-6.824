# Lecture 3:Big Storage
Constructing a distributed system is mainly about constructing a distributed stroage system.

**Why Hard**\
Performance -> Shading -> Fault Tolerace -> Replication -> Inconsistency -> Low Performace

For a ideal Strong Consistent model, we want it to perform like one single server which has only one data and do only one thing at a time.As for a storage sys, answering request means modifying the stored data.To make the service predictable, we define a rule: answering to one request at a time, so that every request can see the latestly modified data.\
*eg:c1 make the X be 1,at the same time,c2 make the X be 2,so what we get when we read X?*\

What's more, when we have several replicas, this problem could be harder.\
Here is a bad design for multi-replica-sys:\
There are 2 servers both containing a replica for all data.They store a key-value list in their disk.We want them to be totally consistent,which meas each writing request will be executed in 2 servers and each reading request can be merely executed in 1 server.Then there comes a problem that if c1 request to make X to be 1,c2 request to make X to be 2.Both requests will be sent to 2 servers and we don't do anything to make the servers execute both request at a same order,so the consistency can't be guaranteed.\
Although this problem is solvable, but more corespondance will be needed and complexity will be increased.There is a whole range of solutions to obtain good consistency with some acceptable errors.\
