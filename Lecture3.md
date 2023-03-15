# Lecture 3:Big Storage
Constructing a distributed system is mainly about constructing a distributed stroage system.

**Why Hard**\
Performance -> Shading -> Fault Tolerace -> Replication -> Inconsistency -> Low Performace

For a ideal Strong Consistent model, we want it to perform like one single server which has only one data and do only one thing at a time.As for a storage sys, answering request means modifying the stored data.To make the service predictable, we define a rule: answering to one request at a time, so that every request can see the latestly modified data.\
*eg:c1 make the X be 1,at the same time,c2 make the X be 2,so what we get when we read X?*

What's more, when we have several replicas, this problem could be harder.\
Here is a bad design for multi-replica-sys:\
There are 2 servers both containing a replica for all data.They store a key-value list in their disk.We want them to be totally consistent,which meas each writing request will be executed in 2 servers and each reading request can be merely executed in 1 server.Then there comes a problem that if c1 request to make X to be 1,c2 request to make X to be 2.Both requests will be sent to 2 servers and we don't do anything to make the servers execute both request at a same order,so the consistency can't be guaranteed.\
Although this problem is solvable, but more corespondance will be needed and complexity will be increased.There is a whole range of solutions to obtain acceptable consistency with some acceptable errors.

**Goal of GFS**
* GFS do lots to fix the above problem.
* What they were looking for one goal is **Big** and **Fast**.
* They also wanted a file sys that was sort of **global** in the sense that many diff app could get a it.
* In order to get bigness and fastness they need to **split** the data.Every file will be automatically splited and saved at many servers,which makes read and write faster and make it available to save big data.
* Because we sere built on numerous servers we want **automatic recovery**.
* GFS was designed to in a **single data center**.
* GFS was disigned for **internal use**.
* It was tailored in a number of ways only for **big sequential file** read and writes.
* The paper proposed a fairly heretical view that it was okay for store system to have a **weak consistency** to obtain a better performance.

**Architecture**
* A GFS cluster consists of a single *master* and multple *chuckservers*.
* Files are divieded into fixed-size *chunks*.Each chunk is identified by an immutable and globally unique 64 bit *chunk handle* assigned by the master at the time of chunk creation.Chunkservers store chunks on local disks as Linux files and read or write chunk data specified by a chunk handle and byte range.For reliability,each chunk is replicated on multiple chunkservers,default is 3.
* The master maintains all file system metadata,which includes the namespace,access control information,the mapping from files to chunks,and the current locations of chunks.It also controls system-wide activities such as chunk lease management, garbage collection of orphaned chunks, and chunk migration between chunkservers.The master periodically communicates with each chunkserver in HeartBeat messages to give it instructions and collect its state.

**Master Data**\
It's got 2 main tables we care
1. one table that maps file name to an array of chunk IDs(or chunk handle)
2. second table that maps each chunk handles to a bunch of data about that chunk
   1. the list of chunkservers that hold the replicas of the data
   2. the version number for each chunk
   3. which chunkserver is the primary and also the expiration of time of the lease(about primary),cuz all writes to a chunk have to be sequence of the chunks primary.
   
This stuff so far it's all in RAM,and just be gone if the master crashed.So in order to reboot the mater and not forget everything about the file sys,the master actually stores all of this data on disk as well as in memory.So reads are from memory but writes,above data reflected on this writes have to go to the disk.\
And the way to actually manage that is the master has a log on disk and everytime it changes the data it appends an entry to the log on disk and checkpoint.And some data above needn't to be on disk like the lisk of chunkservers(master will talk to each chunkservers after reboot),information about primary.\
However if the master crashes and has to reconstructed its state you wouldn't want to have to reread its log file back starting from beginning of the time when the server is installed maybe a few years ago,so the master sometime checkpoints its complete state to disk which takes some amount of seconds of a minute.And then it only need to plays just the portion of a log that starting at the checkpoint.

**Steps of Read and Write**\
**Read**\
The application has a file in minds and an offset in the file that it wants to read some data from,so it sends the file name and the offset to the master,and the master looks the filename and use the offset diveded by 64 megabytes to find that chunck,in its chunktable finds the list of chunkservers have replicas of the data returns the client.
1. client sends file name and the offset to the master 
2. master sends the chunk handle *H* and the list of servers,which are cached by client for probable repeated read.
3. the talk to one of the chunkservers,tells the chunk handle and offset.Chunkserver find the desired chunk and range of bytes and return it to client

**Write**\
Client ask master look to add the end of this file
* If No Primary:\
  1.master find up-to-date replica,(up-to-date means the version of the replica is equal to the version number the master knows is the newest number.we can't just find the maxium version number cuz some chunkservers may be offline but keep the newest replica)
  2.pick one to be primary, others to be secondary.
  3.increments the version number and write that to disk.
  4.tells chunkservers who are primary or secondary,and the new version number,all are written to the disk of chunkservers









