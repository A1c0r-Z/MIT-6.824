# Lecture 4: Fault Tolerence and Replication and VMware FT
The topic for fault tolerence provide high availability that is you want to build some server even have some hardware problem,we can still provide service.
And the tool we use is replication.

**So what kind of failure can replication deal with?**
* The easiest way to characterize the failure is fail stop failures of single computer.fail stop is a sort of generic term and fault tolrence is that if something goes wrong would say the computer,
the computer would simply stop executing.Like someone unplug your power cable or your connection to network,fan on server break and cpu will be off.
* But no bugs.
* There is also some limits to replication too like we were really assuming that the primary and backup are independent.If they have correlated problem,replication can't help us.


The beginning of the paper mentions a couple of different approaches of replications:
1. state transfer:the primary sends a copy of its entire state that is for example the contents of its RAM to the backup.
2. replicated state machines:they just send the external events like arriving input from outside world.we believe that if 2 computer see the same input in the same oreder at the same time,thy will be the same.

The reason why people tend to latter one is that it usually send smaller message.

The paper only deals with processors,and it's no clear to how to extend to a multi-core machine where the interleavings of the instructions from the two cores are non-deterministic.So we no longer have this situation on a multi-core machine, where we just let the primary and backup to execute and they're all else equal,they're not going to be the same cuz they won't work on multiple cores.VMware since come out with a new possibly completely different replication system that does work on the multi-core,which apears to 'me' using state transfer instead of replicated state machines,cuz it's more robust,and also more expensive.

**So if we want to build a replicated state machines,we got number of questions to answer.**
1. We need to decide what level the replicate state/what do we mean by state. Todays paper(VMware) has a really intersting answer for this,it replicates full states of machine that is all memory and all machine register,a very very detailed replicatoin scheme,just no difference.It's quiet rare,cuz most replication scheme out there go GFS route for effiency.
2. we have to worry about how closely synchonized the primary and backup have to be.cuz it's likly primary precede,and having backup acutally executes really in the same as primary is expensive,so what we focused on is how close the synchonization is.
3. If the primary fails,there have to be some scheme to switch over and the client know it shouldn't talk to old primary.
4. There are anomalies when cut over and we need to cope with them,It's almost impossible to design a cut over system in which no anomalies are ever visble.
5. We will need to create new replicas if one fails forever, by state transfer.

**VMware FT**\
VMware is a virtual machine company selling virtual machine technology.
What paper is about we got at least 2 physical machines running their virtual machine monitors and os over it and app over os, which are exactly the same,assuming that there's network connecing these two machines,and in addition to this networkd,some sets of clients are sending request.Actually VMware didn't use local disk,but using disk server connected to network.

* Client sends a request to primary that generate an interrupt.And this interrupt acutally goes to virtual machine monitor in the first instance,vitual machine monitor sees a input for this replicated service and so it does 2 things,(1)stimulates a network packet arival interrupt into the primary guest operationg system,(2)send back out to the network a copy of that packet to the backup virtual machine monitor and backup does (1),process in the same way and stay synchronized.
* Of course,the servie will reply to the client on the primary,service will generate a reply packet and send it on the NIC which virtual mahcine emulating,and virtual machine monitor will see the packet and will actually send the reply back out the network.Because backup running the same thing,so i.It also generates a reply packet which is sent to the emulated NIC,and vitual machine monitor will see it and drop the packet cuz it knows this is backup which is not allowed to generate output.
As far as termiology goes, paper call this stream(from primary to backup) of input events *“Log Channel”*,and the event called *"Log Entries"*
* Where the Fault Tolerence comes in is that the primary crashes,what the backup is going to see is that it(backup) stopes getting stuff, stops getting log entries.And it turns out that the backup can expect to ge many per second because one of the things that generates log entries is periodic timer interrupts in the primary,each interrupt generates a log entries into the back up.These timer interrupt gonna happen hundreds each of second.If the primary crashes,the vitural machine monitor will realize that.In that case,the paper says that the backup goes alive,which means that it stops waiting these input events on the logging channel from the primary,and instead the vitual machine monitor just let this backup execute freely.VMM also let the client requests goes to backup instead of the primary and the VMM stop discarding the backup's output.Now the backup directly gets the inputs and sends out ouputs.

**Non-Deteministic Events**\
We have been assuming that as long as the backup sees the packet from the client it'll execute identically to the primary and that's actually glossing oversome huge and important details.\
So one problem is there are some things that are non-deterministic.\
1. External Input.They just arrive whenever they arrive,they're not predictable.The only form of input and output in the system is network packets which consists of for us is the data in the packet plus the interrupt signalling that the packet had arrived.So when the packet arrives, ordianarily the NIC DMA makes packet contests into memory and then raises an interrupt which the os feels.And the interrupt happens at some point in the instruction stream.Both of those have to look identical in the primary and backup or else some execution may diverge.So the real issue is when the interrupt occurs exactly, at which instruction the interrupts happen to occur and better be the same on primary and backup.So the key is the content of the packet and the timing of the interrupt.
2. Weird Instructions.There's a few intructions beahave differently on different conputers depending on something like there's maybe a rondom number generator.
3. Multi-core Parrelism.

It's these events that go over the logging channel and so the format of a log entry was't said,but "I" am guessing really three things in it.
1. Instruction number.It means the the number of instructions since the Machine booted.
2. Type.
3. Data.For fake the same result of some intruction in backup.

Eample:Assuming the hardware in this case emulated hardware virtual machine has a timer that ticks say a hundred times a second and causes interrupts to operating system.The timer on the physical machine ticks and delivers an interrupt to VMM on the primary, the VMM at appropiate montment stops the execution of the primary, writes down the instruction number that it was at since machine booted and then deliver sort of fakes simulates and interrupt into the guest operating system on the primary at that instruction number says you're emulating the timer, just ticks.Then primary virtual machine monitor sends the interrupt that instruction number to the bakcup.There's also a physical timer on backup but not giving them.Whend the log entry interrupt,the backup VMM will arrange the special CPU to cause the physical machine to interrupt at the same instruction number and then VMM gets control again from guest and then fakes the timer interrupt to the backup operating system.

**Output Rule**\
What the output in this system means sending packets.\
The real case may be little complicated then was said above.\
Supposing that we were running a some sort of database server, and the client operation that our database server support is incremnt.So let's just say everthing is fine so far and both primary and backup have value 10 in memory.Some client send an increment request to primary.Suppose the primary does indeed generate the reply here back to the client but the primary crashes just after sending its reply to the client and much worse it turns out that the logging entry and channel got dropped also.So now the client receive 11 but backup still get 10.So now there's another increment and backup will increase the number cuz primary died,and the reply will still be 11.So this is a disaster.\
The solution to this problem is ouput rule.\
The idea is that the primary isn't allowed to generate any ouput untill the backup acknowledges that it has received all log records up to this point.\
So the real sequence is that the input arrives and primary sends a log entry restrictfully before it sending the ouput.And the ouput won't be sent untill backup acknowledges that it got the packet(it don't need to really execute it before sending the acknowledgement.\
The primary has to delay at this point waiting for the backup to say that it's up to date.This is a real performance thorn in the side of just about every replication scheme.We can't get primary get too far ahead of the backup becasue if the primary fail,it will mean the backup lagging behind the application.

**Duplicated Output**\
One possibility is that the situation maybe the primary crashes after its output is released, so the client does receive the reply then the primary crashes, the backup input is still in the event buffer. When the backup goes alive backup has to consume all the log records lying around that has not been consumed to catch up to the primary otherwise, it won't take over the same state.So for the example, the backup will increase the number to eleven and reply to the client,so now the client get two reply,which is also anomalous.\
But the good news is, almost certainly the client is talking to the service using TCP, when the backup takes over the backup since the state is identical to primary when it generates the TCP packet, it will generate the same TCP number as an original packet and the stack on the client will know that's a duplicate packet which will be discarded, and the user will never see the duplicate packet.

**Test-and-Set Service**\
Up to now, we have been assuming that the machines are fail-stop.\
But another very common situation is if the 2 machines are still up and running and executing,but there is something funny happen in the netwrok that causes they can't talk to each other but can still talk to clients.So if this happen,the 2 machine will think the other one crash,and make itself go alive and diverge finally.\
And the way the paper solve it is by apealing to an outside authority to make the decision about which of the primary and the backup is allowed to be live.It turns out the storage on some external disk server which happen to abort this test-and-set service.So if one want to go alive,it will send test and set requests to the server.So the second one will know there have been a primary,and will not go alive.

One of like deep rules in mit6.824 is that you cannot tell whether the computer is dead or not, all you know is that you stopped receiving packets from it.
And the test-and-set service is acutally single-point-of-failure,this is kinda disappointing.










