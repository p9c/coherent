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

