# Lecture 1

**Motivation**:
1. high-computation/parallelism
2. fault tolerence *eg: with two doing same job,if one goes wrong,we can use the other one*
3. physical reason: some problems are distributed naturally and physically *eg: two banks,one in London,one in NY*
4. security/isolated *eg: if you have some untrusted code,you may want it run in another mechine so that the error won't impact your machin*

**Challenge**:
1. concurrency
2. partial failure
3. performance: to make the performance reach your expectation

High level goal of distributed sys is to get what people call **scalable** speed-up.\
**Scalability**:if I have some problem I solve with a computer and I get an another computer,I can solve the problem in half of the former time.\
Cons:We can use more hardware for higher performance instead of hiring coder to recode the program which could be more expensive.

**Fault Tolerence**:there are always some problems somewhere in your distributed sys,big scale turns the problems from very rare events into constant problem, so the failure handling ability is really needy.
1. **Avalability**:some well designed sys can continue to work even if some **certain** kind of error happen.*eg: using replicas,if one failed,use another one*
2. **Recoverability**:if something goes wrong maybe the servers will stop working and wait for someone to repair,after which can work like nothing bad gone wrong
