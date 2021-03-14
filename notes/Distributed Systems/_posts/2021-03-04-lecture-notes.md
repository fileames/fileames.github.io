---
layout: note
title: MIT 6.824 Lecture Notes
categories: notes 
permalink: notes/distributed-systems/lecture-notes/

hidden: true
---

- toc
{: toc }

## Lecture 2: RPC and Threads Notes
Lecture 2 Notes for [MIT 6.824 Distributed Systems](https://pdos.csail.mit.edu/6.824/index.html){:target="_blank"} course.

**Race Conditions**  
-race flag to detect race conditions
{% highlight bash %}
$ go run -race temp.go
{% endhighlight %}

Two ways of coordination:
1. Channels
- Useful when no data sharing between threads
2. Locks and Condition Vars
- Shared variables

When main returns and other threads don't, garbage collector does not release the local variables of main since the threads have reference to them.

WaitGroup to keep track of the returned threads:
{% highlight go %}
var done sync.WaitGroup
done.Add(1) // Increase the number of threads to be returned.
done.Done() // Called when a thread returns
done.Wait() // Waits for the Done() for the number of times by decided by Add()
{% endhighlight %}

**RPC semantics under failure**
- at least once:	client automatically retires until response
- at most once:	(duplicates) server makes sure it doesn't execute twice (go)
- exactyl once:	hard! 

> **Failures** make PRC not equal to PC  

## Notes From Google I/O 2012 - Go Concurrency Patterns
[Slides](https://talks.golang.org/2012/concurrency.slide){:target="_blank"} - [Talk on Youtube](https://www.youtube.com/watch?v=f6kdp27TYZs){:target="_blank"}

**Goroutines**  
It's very cheap. It's practical to have thousands, even hundreds of thousands of goroutines.  
It's not a thread.  
There might be only one thread in a program with thousands of goroutines.   
They are multiplexed dynamically onto threads as needed to keep all the goroutines running.  

**Channels**
{% highlight go %}
// Declaring and initializing.
var c chan int
c = make(chan int)
// or
c := make(chan int)
// Sending on a channel.
c <- 1
// Receiving from a channel.
// The "arrow" indicates the direction of data flow.
value = <-c
{% endhighlight %}

Sending and receiving are both blocking so synchronization.
Buffering removes synchronization. 

**Patterns**
- Generator (function that returns a channel): Channels as a handle on a service, we can have more instances of the service.
- Multiplexing: Use fan-in function to combine two or more channels. 
{% highlight go %}
func fanIn(input1, input2 <-chan string) <-chan string {
 c := make(chan string)
 go func() { for { c <- <-input1 } }()**
 go func() { for { c <- <-input2 } }()**
 return c
}
{% endhighlight %}

**Select**
- All channels are evaluated.  
- Selection blocks until one communication can proceed, which then does.  
- If multiple can proceed, select chooses pseudo-randomly.  
- A default clause, if present, executes immediately if no channel is ready.

Fan-in with select:
{% highlight go %}
func fanIn(input1, input2 <-chan string) <-chan string {
	c := make(chan string)
	go func() {
		for {
			select {
				case s := <-input1:  c <- s
				case s := <-input2:  c <- s
			}
		}
	}()
	return c
}
{% endhighlight %}

To not wait for too long:
{% highlight go %}
case <-time.After(1 * time.Second):
	fmt.Println("You're too slow.")
{% endhighlight %}

Quit channel on main:
{% highlight go %}
quit := make(chan bool)
c := boring("Joe", quit)
for i := rand.Intn(10); i >= 0; i-- { fmt.Println(<-c) }
quit <- true
 ----
case <-quit:
 return
{% endhighlight %}

Two way communication on quit channel can be used when goroutine has some cleanup needed to be done, main waits until a message from the goroutine on quit channel before returning.

**Q:** How do we avoid discarding results from slow servers?  
**A:** Replicate the servers. Send requests to multiple replicas, and use the first response.

## Lecture 3: GFS
Storage is hard. Why?  
- High performance -> Shard data across servers
- Many servers -> Constant faults, crash once a year 1000 machines -> 3 failures a day
- Fault tolerance -> Replication
- Replication -> Inconsistencies
- Strong consistency -> Lower performance  

Ideal Consistency: Behave as a single machine. This is hard because of concurrency and failures.  

**GFS**  
Non-standard for that time:  
- One master (Why have single point of failure?)
- Can have inconsistencies.  

GFS has to be:  
- Big: Large data set
- Fast: Automatic sharding
- Global: All apps see the same file system
- Fault tolerant: Automatic  

**Design**    
Master stores changes on logs then responds to clients. This way if master fails in between client does not see strange results.

##  Lecture 4: Primary-Backup Replication
**Failures**  
  
Fail-Stop: Failure stops the computer, does not produce weird results  
Replication won't solve logical bugs, configuration errors, malicious.  
Can be handled: earthquake kind of disasters. If primary and backup are on different places and one is unaffected replication could help.  

**Challenges**
- Has the primary really failed? Could be a network partition. Want to avoid split-brain with two primaries.
- Keeping primary-backup in sync, dealing with non-determinism
- Fail-over. Failure in the middle of the operation, what to do? Multiple backups, which one to choose?  

**Two Approaches**
1. State Transfer: Transfer state changes. If operation generates a lot of state this can be expensive.
2. Replicate State Machine (RSM): Transfer operations  

**Level of Operations to Replicate**
- Application level: Apps should be modified.
- Machine level: Operations are ordinary computer operations. Application, OS do not have to be modified. Transparent. Can be done with hardware replication but also with virtual machines.  

**VM FT**  
[Fault-Tolerant Virtual Machines (2010)](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)  
Any events, external interrupts to the Vm is first captured by the hypervisor. On the paper, hypervisor not only sends it to the virtual machine but also to a logging channel to a backup.

## Lecture 5: Fault Tolerance: Raft (1)
2f+1 servers must be running for f+1 to be majority.  
**Why logs:** Order, retransmission, persistence, space for tentative operations.  
**Consensus algorithms** allow a collection of machines to work as a coherent group that can survive the failures of some of its members.

**Replicated State Machines**:
Replicated state machines are typically implemented using a replicated log. Each server stores a log containing a series of commands, which its state machine executes in order. 
*Keeping the replicated log consistent is the job of the consensus algorithm.*  
  
**Raft**  
Main goal of Raft: **understandability**. Paxos was hard to understand.  
3 states: *leader*, *follower*, *candidate* (used to elect a new leader)  
In normal operation there is exactly one leader and all of the other servers are followers.  

Raft divides time into random length terms. Each term starts with an election. 
   
If a candidate wins the election, then it serves as leader for the rest of the term. In some situations an election will result in a split vote. In this case the term will end with no leader. Then new election on new term.

Communication by RPC:
- **RequestVote**: By candidate on election
- **AppendEntries**: By leaders to replicate log entries and to provide a form of heartbeat  

If a follower receives no communication over a period of time called the *election timeout*, then it assumes there is no viable leader and begins an election to choose a new leader.  
  
Nice visualization: [Secret Lives of Data](http://thesecretlivesofdata.com/raft/){:target="_blank"}
