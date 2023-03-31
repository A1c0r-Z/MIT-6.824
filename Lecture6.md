# Lecture 6: Raft(1) 

* Mapreduce replicates computation,but that is controled by a single master.
* GFS replicates data,but relies on a single master to choose primary.
* VMware FT replicates computation write on a primary and a backup,if fail,it relies on a single test-and-set server to help chosse which backup take over.
They all have a single entity required to make a critical decision about who is the primay.
Very nice thing to have a single entity decide is that it can't disagree itself
