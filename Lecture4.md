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
2. replicated state machines:they just send the external events like arriving input from outside world.we believe that if 2 computer see the same input in the same oreder at the same time,thy will be the same.

The reason why people tend to latter one is that it usually send smaller mesaage.

The paper only deals with processors,and it's no clear to how to extend to a multi-core machine where the interleavings of the instructions from the two cores are non-deterministic.So we no longer have this situation on a multi-core machine, where we just let the primary and backup to execute and they're all else equal,they're not going to be the same cuz they won't work on multiple cores.VMware since come out with a new possibly completely different replication system that does work on the multi-core,which apears to 'me' using state transfer instead of replicated state machines,cuz it's more robust,and also more expensive.

So if we want to build a replicated state machines,we got number of questions to answer.
1. We need to decide what level the replicate state/what do we mean by state. Todays paper(VMware) has a really intersting answer for this,it replicates full states of machine that is all memory and all machine register,a very very detailed replicatoin scheme,just no difference.It's quiet rare,cuz most replication scheme out there go GFS route for effiency.
2. we have to worry about how closely synchonized the primary and backup have to be.cuz it's likly primary precede,and having backup acutally executes really in the same as primary is expensive,so what we focused on is how close the synchonization is.
3. If the primary fails,there have to be some scheme to switch over and the client know it shouldn't talk to old primary.
4. There are anomalies when cut over and we need to cope with them,It's almost impossible to design a cut over system in which no anomalies are ever visble.
5. We will need to create new replicas if one fails forever, by state transfer.

**VMware FT**
VMware is a virtual machine company selling virtual machine technology.

















