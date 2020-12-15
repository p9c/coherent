# apiweb
A plan9 like model for an operating system and kernel design for next generation, seamlessly distributed computing

##### Loki Verloren
Novi Sad, December 12, 2020

## The Inspiration

In the process of building ParallelCoin Plan 9 from Crypto Space Hard Fork, in order to facilitate my debugging of concurrent code I wrote a logging library including code location based path filters and eventually a channel-based logger that sends log entries to the process that started it up. 

Attached to that for reasons of necessity - the Windows operating system has no native concept of process signals - the simple logging API had a quit command built into it and then it became clear it needed to be the primary mechanism for keeping control of these little trees of processes. 

The ParallelCoin Pod has 5 components:

1. GUI interface
2. Blockchain Node
3. Wallet
4. Miner
5. Miner Worker

The GUI runs the next three, and the miner runs the last. The three in the middle also print logs through to their parent processes, where they can also be directed via the same channel mechanism to populate a table to generate a GUI log view. 

The miner worker does not use the same mechanism exactly, instead it has a net/rpc API with which it is given the gossip network channel key, delivered a new block header to hash, told to pause or shut down. The net/rpc library is inferior to the home-made SimpleBuffers codec, pipe-worker library, as it is not simple to multiplex. The current version uses a map of magic bytes with the handler for the payload that goes with it. Both sides of the channel can send at the same time as receive, it is full duplex, unlike the client-server model of net/rpc.

### Interface Slices for Serializer Types

The SimpleBuffers codec is an extremely simple serialization codec that aims to allow the progressive and selective decoding of the message, wherever possible, without making any copy of the data. In its current, initial incarnation, bytes are decoded to integers, strings are most likely copied, but it is possible to short-cut this copy/decode process using 'unsafe' libraries, that I have not delved into due to it's well deserved name. However when all else is ready it will be streamlined maximally.

Simplebuffers works by assembling strings of codec types (each in their own package) that handles converting the data into bytes and back into variables. The difference is that when it is decoded, the location of each element is known by its byte position and by its definition thereby, can be determined its size. It is not deeply exercised in the current codebase, but with larger data structures, selective encoding can be helpful to the design of the application's latency performance.

Some elements of the data may be, as always exists in peer to peer networks, information that should be relayed as quickly as possible. In this way this data could be decoded first, then sent to the relay, and then a further process with more exhaustive use of the data can decode the un-decoded parts, and the codec also keeps the decoded forms alongside the original raw data and subsequent accesses get the decoded values directly.

In addition to being able to encode and decode arbitrary strings of variables into binary and back, a codec has to contend with issues relating to versioning, as well as allowing applications to inspect the data structure without necessarily having the code available to process it.

Schema are also just a very specific type of data structure in a form similar to programming code, with grouping and separation operators, like the frames that get filled with walls and windows and whatnot, or like the layout boxes of a visual markup language. These schema start with a header that includes the URLs for locating libraries that process the type of data in a message as well, both source code and potentially compiled binaries, secured by asymmetric cryptography.

### Protocol

Schema are exchanged in the initial handshake between two nodes in the apiweb. Both sides then can register each other's set of services, and presumably both through polling and subscription, updates to APIs at an address can be requested. This initial process boils down to simply unmarshalling a stored form of the peer's API schema once the client has all interesting schema from the server. Note that this protocol is concurrent, both sides stream messages to each other across each side of their duplex connection without any coordination.

The transport can be any kind of duplex channel. Stdin and Stdout and pipes to them between parent and child processes, peer processes can open rendezvous named pipes, peers separated by network connections use Reliable UDP (using a scalable Reed Solomon message segmentation scheme), or TCP or HTTP, any mechanism that allows bidirectional streaming of data can interpose between two processes.

Services are identified by the source code repository URL where they originate, combined with a network location specification. When two processes handshake, they compare their network locations and if they discover they are on the same machine, they create a named pipe, if they are in the same lan subnet same gateway, they open a high speed/low latency connection, if they are in separate networks, they send packets through their local router to find the router of the other party.

Schema have a DAG of versions like a Git repository, and part of the versioning difference data is the mapping of exact correspondences between variables in one version and the ones that follow and precede it. In this way, the pain of versioning can be reduced because old clients still can understand that some specific set of fields that used to be in one location can be found in the new one for the same data type in new locations. 

In this way the nodes can incrementally update at their own pace according to the needs of their users, without being cut off from the network all at once, more often the opposite - that the benefits to upgrade are clear and easy to see. This feature is to reduce the pressure on deployment teams and help bring a more incrementalist, modular methodology to their systems development and administration. 

There is plenty of functional, old systems out there in daily use, for reasons more than just even economics, sometimes security models haven't really changed, sometimes there is just too many who can work the old system and too few for the new. Like QWERTY keyboards and secretaries, who are notoriously averse to learning new technology for their non-technology business. We have 20 flavours of Proof of Stake but Proof of Work still has some big advantages due no way to cheat limited supply of computing hardware.

### Interfaces

Nothing much of the foregoing really steps far forward but in the user interfaces we are doing something somewhat unique. User interfaces are APIs, consisting of linear threads of execution painting pixels, triggering devices outputs, reading inputs making models that map the inputs to visual matrixes of widgets, little bounded areas of the total display.

The interfaces in Web Browsers are similar, with static resources, server side and client side programs that perform relevant tasks to data. The Gio GUI library that I have been using to build the GUI for the new ParallelCoin software is written from the base up to be utilised best with a functional/imperative coding style, the native Go idiom.

Its native operations, both outputs (painting) and inputs (pointers, gestures, keys) boil down to structured streams of data that control the operation of generalised processes, can be serialized and passed across channels between nodes to convey GUI interface layouts, and, this is the exciting part - those descriptions carry the data for populating the forms and layouts with content based on URLs that are the previously mentioned APIs (query and response pairs), so a client downloads the interface, and then each of those URLs is queried and the data rendered in the interface.

Every single element of an interface could conceivably, therefore, be populated with data fetched from a different physical machine, as equally as from one. There isn't any code to run on the client side apart from rendering data. As the chains of dependencies can be discovered and analysed, it then becomes possible for a node to then acquire the processing libraries, and with those process the raw data that the higher level endpoints would provide, at lower latency, by eliminating the additional delays caused by receive-process-send chains with high transit time cost.

It could not even in the vaguest way be compared to or connected with artificial intelligence, however. The phylogeny of message versions are incrementally but precisely connected to each other. The discovery of the possibility of shifting processing load to a local system is completely deterministic. Servers could even automatically alert their client nodes to the possibility of consuming more raw data and less waiting for processed data to move around, nodes then sync their code caches, start pulling the less refined source data and producing the user's desired output sooner, closer to their position. Such locality-seeking also has natural security benefits, as information doesn't have to travel across so many open relays.

### Connection to Plan 9 operating system.

Plan 9's architecture includes the concept of turning everything into a server. This was relatively vague and obscure back in the times it was invented, but nowadays everyone calls them "API" or "RPC" or some similar related concept. 

Key to enabling this through and through design with everything functioning as servers with various ways to reach them and is the routing logic is closer to the edge of the network. Each device has a primary server that acts as the local machine root, tied then to user and then process invocation. Access is defined by control of asymmetric authentication keys, and should of course be extended with a subkey delegation mechanism, so that the more powerful the key, the less places it needs to be watched, and vice versa. 

Checking authorisation between two communicating processes is done via the same kind of mechanism as found in SSH, but extended through the use of subkeying as found in HD keychains and delegation schemes in many cryptocurrency protocols. Each resource has a set of authorised public keys attached to it, which are unlocked by proving control of the secret key with signatures on messages.

### Kernel

Preemptive multitasking has taken over because of its ability to forcibly correct programming errors. If code causes resources to be exhausted, in a preemptive multitasking system, the kernel can intervene, stop the running process and allow other processes to resume access to them.

Cooperative multitasking depends entirely on the programmer ensuring that when their program is finished with a resource, that it gives it back. 

It helps to refer to processing time as a 'resource' comparable to, for example, frames, pixels, bytes, seconds, or any other means of dividing up capacity. Then you can recognise the similarity between the problem of multitasking on the temporal axis on a very limited number of cores with a limited number of cycles in that time.

Just as the ParallelCoin 9 way parallel block interval remodels the idea of a clock to include a variable schedule, as I observed, it could even be changed to be the progressive values of a random number generator with the seed baked into the genesis block. However, the parallel intervals are better because it forms a connected, elastic 9 body interaction. As any student of physics would be aware, three bodies moving in space affected by both in a mutual interaction produces an endlessly random though recognisable set of phase states distributed through the timeline of the process. 

In terms of combinatorics, the three body problem is no less complex even in two dimensions. Once you have three factors moving in relation to each other, every moment of time and every element has 6 possible vectors, 3 real, 3 imaginary. On the subject of imaginary numbers, they are numbers with the mirror image of arithmetic operations in both time and sequence. Squaring i's roots them, multiplying, divides, and so on and so forth.

The conventional kernel/userland divide is exactly containing the problem within a two body problem. The downside of this approach grows with the number of processing units and the amount of latency of signal between them (usually dictated by spatial position).

The purpose of a kernel is to facilitate the sharing of resources under the control of the CPU so that no process attempts to use a resource at the same time in contradictory ways. Writing different data to the same location of screen, memory, or disk, for example. This is of course the nature of a Von Neumann Engine.

However, in physical hardware, there is an intrinsic level of distribution, of latencies and bandwidth at various scales of distance, and programming even a single CPU die at modern CPU clock speeds results in not just asymptotic rise of resistance, but the need to use synchronisation mechanisms, where previously the time of travel over many relay nodes in the circuit starts to unravel the ivory tower idyllic image of a central processing unit. Just as the atom has increasingly not been proving to be atomic.

### Threads of Execution

In modern 64 bit CPUs there is enormous amounts of the fastest, densest computer memory that exists. The Ryzen processors of the previous generation at writing have 19mb divided up into 3 tiers each one exponentially larger than the next.

Instead of having one kernel in the operating system, each processing thread runs a tiny little kernel, implementing an adaptive scheduling scheme depending on the workload.

Conventional modern operating systems, micro and monolithic kernels both now have 1000hz preemption schedules, a fixed duration, which was 100hz up until about 15 years ago. The value is settled upon as a compromise and interpolation between all use cases or kept uniform where use cases override the raw economics. It sped up because displays got faster, input devices got faster, and network bandwidth grew and latency shrank.

But in fact, there is multiple easy to demonstrate cases where 1000hz is too slow.

Picture a snaking chain of network cables spaced apart at their break even point distance (eg ~100m for CAT-5 STP of typical grade and gauge of copper). Ethernet cables give a good illustration. From pole to pole is approximately 10,000km, or 100k of these little daisy chains improbably stretched across the globe.

Thus, if the time to relay the signal between each of those nodes along the path is 1ms, or 1/1,000th of a second, then that's 100,000 * 1,000 or 100 million milliseconds, which is 100,000 seconds, 1666 minutes, 27.777 hours, or a bit more than a day.

> ## sticking to von neumann model eventually escalates to the absurd

So, instead of a big fat kernel, we have a little kernel for each thread of the CPU, and they attempt to efficiently prune the tree of waiting processing tasks based on memory distance (which equals latency and contention) where their data requests go.

At the next layer, we have the device kernels. these demand the highest priority and interact with each kernel using interrupts to monopolise the processor at the moment the data is ready to process. These also migrate across cores also in a way that aims to batch memory accesses more for the processes that are using them. Thus it is a simple heuristic that code and working data will be placed near each other. 

At the micro-level, memory pages of 4kb in size, have a time cost of relocation between cache levels and memory. Processing cannot take place until the smallest caches are populated with working data, so each of the kernels will thus share different parts of the cache levels, divided up pretty much evenly. Instead of shoehorning everything into the 1ms time window of modern preemptive multitasking operating systems, processing is allocated to the most proximate first, which will be based on proximity and total bytes throughput.

### Opportunistic Randomised Scheduling

Or in other words, processing is opportunistic and instead of one in-between good for nothing specifically value, processes are preempted according to their expenditure of bandwidth. Each process is not given a specific amount of time but a specific amount of bandwidth budget. Moving memory within a cache is the cheapest, between caches is more expensive, to main memory is even more expensive, and to disk, even more expensive, to the cloud, the most expensive.

The only cost for scheduling is then in tracking data throughput. The size of the code segments is immediately available. The operations in the code can be scanned to compute an estimation of the number of times the data will be copied from one place to another, and further refined as it runs, sampling the position within the total executable file, and generating a heat map and computing a cost surface from this.

Some code has extremely tight loops over small pieces of memory, a very good example of this is the treap. With good design, a treap of 4kb can index a considerable amount of data and keep it in order. These searches have a high unit cost of memory copy operations and usually a small amount of arithmetic for index walking. Thus, the scheduler will favour a treap walk operation over a more sparse operation that requires 4x as much memory but half as many read/write cycles, because inside 4kb page of a CPU top level cache, these cycles are thousands of times less than, for example, even just, say, reencoding a text encoding and returning/relaying the result.

By such a scheme of priority, automatically there is a sensible rhythm. A disk read/write cycle of a 4kb page of memory is literally a million times slower on a spinning disk compared to the same page in the fastest, closest caches to the processing cores. So the fast processes will execute thousands of cycles scattered randomly in time, and the slow processes will be also scattered, in the spaces between. The general heuristic is to scatter the smallest the widest, and then allocate into the available space (or time) with gradually bigger pieces.

The result of such a pattern of distribution is automatically favouring the shortest possible time between responses at any given section of the system. The various connection protocol kernels, level 2 of the apiweb kernel, are exactly models of the various connectivity methods available to programs executing on the CPU core. PCI-express, ISA, I2C, ethernet, USB, SATA, memory bus, and so on.

Yes, so each kernel will allocate the time and space in their caches and rebalance it according to its hunger for bandwidth and synchronisation.

For example, spinlocks are a terribly wasteful way to respond to unpredictable realtime input. There is some tasks that require them, but for everything else there is interrupts, triggered when a data transfer between points has been completed. There is still sadly quite some things that require everything to go all the way to the caches, and back out again, but memory, for example, pretty quickly proved to be a good temporary storage location for programs, and opportunistically transferring it in idle time between spurts of work, so-called Direct Memory Access, is slowly but surely taking over as the preferred way to interface slow things with CPUs.



### When in doubt, flip a coin

The second factor in the 'scheduling' system is for resolving when two pieces of code have equal weight to fit into a given moment of time. Just as is done with the select statements in Go, if two items or more arrive in the buffer before the scheduler inspects them, Go's runtime basically flips a coin.

