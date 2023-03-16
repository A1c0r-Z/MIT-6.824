# Lecture 4: Fault Tolerence and Replication and VMware FT
The topic for fault tolerence provide high availability that is you want to build some server even have some hardware problem,we can still provide service.
And the tool we use is replication.\
**So what kind of failure can replication deal with?**\
The easiest way to characterize the failure is fail stop failures of single computer.fail stop is a sort of generic term and fault tolrence is that if something goes wrong would say the computer,
the computer would simply stop executing.Like someone unplug your power cable or your connection to network,fan on server break and cpu will be off.\
But no bugs.
There is also some limits to replication too like we were really assuming that the primary and backup are independent.If they have correlated problem,replication can't help us.

The beginning of the paper mentions a couple of different approaches of replications:
1. state transfer:the primary sends a copy of its entire state that is for example the contents of its RAM to the backup.
2. replicated state machines:they just send the external events like arriving input from outside world.we believe that if 2 computer see the same input in the same oreder at the same time,thy will be the same.\ 
The reason why people tend to latter one is that it usually send smaller mesaage.







