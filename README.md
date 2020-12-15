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

