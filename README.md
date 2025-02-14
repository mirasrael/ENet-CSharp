<p align="center"> 
  <img src="https://i.imgur.com/CxkUxTs.png" alt="alt logo">
</p>

[![GitHub release](https://img.shields.io/github/release/nxrighthere/ENet-CSharp.svg)](https://github.com/nxrighthere/ENet-CSharp/releases) [![NuGet](https://img.shields.io/nuget/v/ENet-CSharp.svg)](https://www.nuget.org/packages/ENet-CSharp/) [![PayPal](https://drive.google.com/uc?id=1OQrtNBVJehNVxgPf6T6yX1wIysz1ElLR)](https://www.paypal.me/nxrighthere) [![Bountysource](https://drive.google.com/uc?id=19QRobscL8Ir2RL489IbVjcw3fULfWS_Q)](https://salt.bountysource.com/checkout/amount?team=nxrighthere) [![Coinbase](https://drive.google.com/uc?id=1LckuF-IAod6xmO9yF-jhTjq1m-4f7cgF)](https://commerce.coinbase.com/checkout/03e11816-b6fc-4e14-b974-29a1d0886697)

_The library is no longer in active public development, all futher work moved to private, this repository turned into read-only mode for historical reference. Thanks to all supporters and contributors._

This project is based on collaborative work with [@inlife](https://github.com/inlife) and inherited all features of the custom [fork](https://github.com/zpl-c/enet) where the native library was heavily modified. You can find the most notable changes [here](https://github.com/nxrighthere/ENet-CSharp/issues/22#issuecomment-432982154). This version is extended, optimized, and involves new features that were not available before to boost the development process and run safely in the managed .NET environment with the highest possible performance.

Although the project is called ENet-CSharp, this is not just a C# wrapper for the native C library, but an independent fork which is incompatible with any other ENet implementation including the [original](https://github.com/lsalzman/enet) since 2.0.0 version. Programmers who are using C/C++ languages can utilize this fork as well in any projects such as [NetDynamics](https://github.com/nxrighthere/NetDynamics).

Features:

- Lightweight and straightforward
- Low resource consumption
- Dual-stack IPv4/IPv6 support
- Connection management
- Sequencing
- Channels
- Reliability
- Fragmentation and reassembly
- Compression
- Aggregation
- Adaptability
- Portability

Please, read [common mistakes](https://github.com/nxrighthere/ENet-CSharp/issues/45) during integration and check [this](https://github.com/nxrighthere/ENet-CSharp/issues?q=is%3Aissue+is%3Aclosed) section before opening a new issue.

Building
--------
To build the native library appropriate software is required:

For desktop platforms [CMake](https://cmake.org/download/) with GNU Make or Visual Studio.

For mobile platforms [NDK](https://developer.android.com/ndk/downloads/) for Android and [Xcode](https://developer.apple.com/xcode/) for iOS. Make sure that all compiled libraries are assigned to appropriate platforms and CPU architectures.

To build the library for Nintendo Switch, follow [this](https://pastebin.com/raw/rbjLgMV2) guide.

Define `ENET_LZ4` to build the library with support for an optional packet-level compression.

A managed assembly can be built using any available compiling platform that supports C# 3.0 or higher.

Compiled libraries
--------
You can grab compiled libraries from the [release](https://github.com/nxrighthere/ENet-CSharp/releases) section or from [NuGet](https://www.nuget.org/packages/ENet-CSharp):

`ENet-CSharp` contains compiled assembly with native libraries for the .NET environment (.NET Standard 2.0).

`ENet-Unity` contains script with native libraries for the Unity.

These packages are provided only for traditional platforms: Windows, Linux, and macOS (x64).

Supported OS versions:
- Windows 7 or higher
- Linux 4.4 or higher
- macOS 10.12 or higher

Usage
--------
Before starting to work, the library should be initialized using `ENet.Library.Initialize();` function.

After the work is done, deinitialize the library using `ENet.Library.Deinitialize();` function.

### .NET environment
##### Start a new server:
```c#
using (Host server = new Host()) {
	Address address = new Address();

	address.Port = port;
	server.Create(address, maxClients);

	Event netEvent;

	while (!Console.KeyAvailable) {
		bool polled = false;

		while (!polled) {
			if (server.CheckEvents(out netEvent) <= 0) {
				if (server.Service(15, out netEvent) <= 0)
					break;

				polled = true;
			}

			switch (netEvent.Type) {
				case EventType.None:
					break;

				case EventType.Connect:
					Console.WriteLine("Client connected - ID: " + netEvent.Peer.ID + ", IP: " + netEvent.Peer.IP);
					break;

				case EventType.Disconnect:
					Console.WriteLine("Client disconnected - ID: " + netEvent.Peer.ID + ", IP: " + netEvent.Peer.IP);
					break;

				case EventType.Timeout:
					Console.WriteLine("Client timeout - ID: " + netEvent.Peer.ID + ", IP: " + netEvent.Peer.IP);
					break;

				case EventType.Receive:
					Console.WriteLine("Packet received from - ID: " + netEvent.Peer.ID + ", IP: " + netEvent.Peer.IP + ", Channel ID: " + netEvent.ChannelID + ", Data length: " + netEvent.Packet.Length);
					netEvent.Packet.Dispose();
					break;
			}
		}
	}

	server.Flush();
}
```

##### Start a new client:
```c#
using (Host client = new Host()) {
	Address address = new Address();

	address.SetHost(ip);
	address.Port = port;
	client.Create();

	Peer peer = client.Connect(address);

	Event netEvent;

	while (!Console.KeyAvailable) {
		bool polled = false;

		while (!polled) {
			if (client.CheckEvents(out netEvent) <= 0) {
				if (client.Service(15, out netEvent) <= 0)
					break;

				polled = true;
			}

			switch (netEvent.Type) {
				case EventType.None:
					break;

				case EventType.Connect:
					Console.WriteLine("Client connected to server");
					break;

				case EventType.Disconnect:
					Console.WriteLine("Client disconnected from server");
					break;

				case EventType.Timeout:
					Console.WriteLine("Client connection timeout");
					break;

				case EventType.Receive:
					Console.WriteLine("Packet received from server - Channel ID: " + netEvent.ChannelID + ", Data length: " + netEvent.Packet.Length);
					netEvent.Packet.Dispose();
					break;
			}
		}
	}

	client.Flush();
}
```

##### Create and send a new packet:
```csharp
Packet packet = default(Packet);
byte[] data = new byte[64];

packet.Create(data);
peer.Send(channelID, ref packet);
```

##### Copy payload from a packet:
```csharp
byte[] buffer = new byte[1024];

netEvent.Packet.CopyTo(buffer);
```

##### Integrate with a custom memory allocator:
```csharp
AllocCallback OnMemoryAllocate = (size) => {
	return Marshal.AllocHGlobal(size);
};

FreeCallback OnMemoryFree = (memory) => {
	Marshal.FreeHGlobal(memory);
};

NoMemoryCallback OnNoMemory = () => {
	throw new OutOfMemoryException();
};

Callbacks callbacks = new Callbacks(OnMemoryAllocate, OnMemoryFree, OnNoMemory);

if (ENet.Library.Initialize(callbacks))
	Console.WriteLine("ENet successfully initialized using a custom memory allocator");
```

### Unity
Usage is almost the same as in the .NET environment, except that the console functions must be replaced with functions provided by Unity. If the `Host.Service()` will be called in a game loop, then make sure that the timeout parameter set to 0 which means non-blocking. Also, keep Unity run in background by enabling the appropriate option in the player settings.

Multi-threading
--------
### Strategy
The best-known strategy is to use ENet in an independent I/O thread and utilize [inter-thread messaging](http://www.1024cores.net/home/lock-free-algorithms/queues) techniques for transferring data across threads/tasks without any locks/mutexes. Non-blocking queues like [Ring Buffer](https://www.slideshare.net/trishagee/introduction-to-the-disruptor) was designed for such purposes. High-level abstractions and logic can be parallelized [using workers](https://forum.unity.com/threads/showcase-enet-unity-ecs-5000-real-time-player-simulation.605656/), then communicate with I/O thread and enqueue/dequeue messages to send/receive data across the network.

### Functionality
In general, ENet is not thread-safe, but some of its functions can be used safely if the user is careful enough:

`Packet` structure and its functions are safe until a packet is only moving across threads by value and a custom memory allocator is not used.

`Peer.ID` as soon as a pointer to a peer was obtained from the native side, the ID will be cached in `Peer` structure for further actions with objects that assigned to that ID. `Peer` structure can be moved across threads by value, but its functions  are not thread-safe because data in memory may change by the service in another thread.

`Library.Time` utilizes atomic primitives internally for managing local monotonic time.

API reference
--------
### Enumerations
#### PacketFlags
Definitions of a flags for `Peer.Send()` function:

`PacketFlags.None` unreliable sequenced, delivery of packet is not guaranteed.

`PacketFlags.Reliable` reliable sequenced, a packet must be received by the target peer and resend attempts should be made until the packet is delivered.

`PacketFlags.Unsequenced` a packet will not be sequenced with other packets and may be delivered out of order. This flag makes delivery unreliable.

`PacketFlags.NoAllocate` a packet will not allocate data, and the user must supply it instead.

`PacketFlags.UnreliableFragmented` a packet will be unreliably fragmented if it exceeds the MTU. By default packets larger than MTU fragmented reliably.

`PacketFlags.Instant` a packet will not be bundled with other packets at a next service iteration and sent instantly instead. This delivery type trades multiplexing efficiency in favor of latency. The same packet can't be used for multiple `Peer.Send()` calls.

#### EventType
Definitions of event types for `Event.Type` property:

`EventType.None` no event occurred within the specified time limit.

`EventType.Connect` a connection request initiated by `Peer.Connect()` function has completed. `Event.Peer` returns a peer which successfully connected. `Event.Data` returns user-supplied data describing the connection or 0 if none is available.

`EventType.Disconnect` a peer has disconnected. This event is generated on a successful completion of a disconnect initiated by `Peer.Disconnect`. `Event.Peer` returns a peer which disconnected. `Event.Data` returns user-supplied data describing the disconnection or 0 if none is available.

`EventType.Receive` a packet has been received from a peer. `Event.Peer` returns a peer which sent the packet. `Event.ChannelID` specifies the channel number upon which the packet was received. `Event.Packet` returns a packet that was received, and this packet must be destroyed using `Event.Packet.Dispose()` function after use.

`EventType.Timeout` a peer has timed out. This event occurs if a peer has timed out or if a connection request initialized by `Peer.Connect` has timed out. `Event.Peer` returns a peer which timed out.

#### PeerState
Definitions of peer states for `Peer.State` property:

`PeerState.Uninitialized` a peer not initialized.

`PeerState.Disconnected` a peer disconnected or timed out.

`PeerState.Connecting` a peer connection in-progress.

`PeerState.Connected` a peer successfully connected.

`PeerState.Disconnecting` a peer disconnection in-progress.

`PeerState.Zombie` a peer not properly disconnected.

### Delegates
#### Memory callbacks
Provides per application events.

`AllocCallback(IntPtr size)` notifies when a memory is requested for allocation. Expects pointer to the newly allocated memory.

`FreeCallback(IntPtr memory)` notifies when the memory can be freed.

`NoMemoryCallback()` notifies when memory is not enough.

#### Packet callbacks
Provides per packet events.

`PacketFreeCallback(Packet packet)` notifies when a packet is being destroyed.

### Structures
#### Address
Contains marshalled structure with host data and port number.

`Address.Port` set or get a port number.

`Address.GetIP()` get an IP address.

`Address.SetIP(string ip)` set an IP address. To use IPv4 broadcast in the local network the address can be set to _255.255.255.255_ for a client. ENet will automatically respond to the broadcast and update the address to a server's actual IP. 

`Address.GetHost()` attempts to do a reverse lookup from the address. Returns a string with a resolved name or an IP address.

`Address.SetHost(string hostName)` set host name or an IP address. Should be used for binding to a network interface or for connection to a foreign host. Returns true on success or false on failure.

#### Event
Contains marshalled structure with the event type, managed pointer to the peer, channel ID, user-supplied data, and managed pointer to the packet.

`Event.Type` returns a type of the event.

`Event.Peer` returns a peer that generated a connect, disconnect, receive or a timeout event.

`Event.ChannelID` returns a channel ID on the peer that generated the event, if appropriate.

`Event.Data` returns user-supplied data, if appropriate.

`Event.Packet` returns a packet associated with the event, if appropriate.

#### Packet
Contains a managed pointer to the packet.

`Packet.Dispose()` destroys the packet. Should be called only when the packet was obtained from `EventType.Receive` event.

`Packet.IsSet` returns a state of the managed pointer.

`Packet.Data` returns a managed pointer to the packet data.

`Packet.Length` returns a length of payload in the packet.

`Packet.HasReferences` checks references to the packet.

`Packet.SetFreeCallback(PacketFreeCallback callback)` set callback to notify the programmer when an appropriate packet is being destroyed. Pointer `IntPtr` to a callback can be used instead of a reference to a delegate.

`Packet.Create(byte[] data, int offset, int length, PacketFlags flags)` creates a packet that may be sent to a peer. The offset parameter indicates the starting point of data in an array, the length is the ending point of data in an array. All parameters are optional. Multiple packet flags can be specified at once. Pointer `IntPtr` to a native buffer can be used instead of a reference to a byte array.

`Packet.CopyTo(byte[] destination)` copies payload from the packet to the destination array.

#### Peer
Contains a managed pointer to the peer and cached ID.

`Peer.IsSet` returns a state of the managed pointer.

`Peer.ID` returns a peer ID.

`Peer.IP` returns an IP address in a printable form.

`Peer.Port` returns a port number.

`Peer.MTU` returns an MTU.

`Peer.State` returns a peer state described in the `PeerState` enumeration.

`Peer.RoundTripTime` returns a round trip time in milliseconds.

`Peer.LastSendTime` returns a last packet send time in milliseconds.

`Peer.LastReceiveTime` returns a last packet receive time in milliseconds.

`Peer.PacketsSent` returns a total number of packets sent during the connection.

`Peer.PacketsLost` returns a total number of lost packets during the connection.

`Peer.BytesSent` returns a total number of bytes sent during the connection.

`Peer.BytesReceived` returns a total number of bytes received during the connection.

`Peer.Data` set or get the user-supplied data. Should be used with an explicit cast to appropriate data type.

`Peer.ConfigureThrottle(uint interval, uint acceleration, uint deceleration, uint threshold)` configures throttle parameter for a peer. Unreliable packets are dropped by ENet in response to the varying conditions of the connection to the peer. The throttle represents a probability that an unreliable packet should not be dropped and thus sent by ENet to the peer. The lowest mean round trip time from the sending of a reliable packet to the receipt of its acknowledgment is measured over an amount of time specified by the interval parameter in milliseconds. If a measured round trip time happens to be significantly less than the mean round trip time measured over the interval, then the throttle probability is increased to allow more traffic by an amount specified in the acceleration parameter, which is a ratio to the `Library.throttleScale` constant. If a measured round trip time happens to be significantly greater than the mean round trip time measured over the interval, then the throttle probability is decreased to limit traffic by an amount specified in the deceleration parameter, which is a ratio to the `Library.throttleScale` constant. When the throttle has a value of `Library.throttleScale`, no unreliable packets are dropped by ENet, and so 100% of all unreliable packets will be sent. When the throttle has a value of 0, all unreliable packets are dropped by ENet, and so 0% of all unreliable packets will be sent. Intermediate values for the throttle represent intermediate probabilities between 0% and 100% of unreliable packets being sent. The bandwidth limits of the local and foreign hosts are taken into account to determine a sensible limit for the throttle probability above which it should not raise even in the best of conditions. To disable throttling the deceleration parameter should be set to zero. The threshold parameter can be used to reduce packet throttling in unstable network environments with high jitter and low average latency which is a common condition for Wi-Fi networks in crowded places.

`Peer.Send(byte channelID, ref Packet packet)` queues a packet to be sent. Returns true on success or false on failure.

`Peer.Receive(out byte channelID, out Packet packet)` attempts to dequeue any incoming queued packet. Returns true if a packet was dequeued or false if no packets available.

`Peer.Ping()` sends a ping request to a peer. ENet automatically pings all connected peers at regular intervals, however, this function may be called to ensure more frequent ping requests.

`Peer.PingInterval(uint interval)` sets an interval at which pings will be sent to a peer. Pings are used both to monitor the liveness of the connection and also to dynamically adjust the throttle during periods of low traffic so that the throttle has reasonable responsiveness during traffic spikes.

`Peer.Timeout(uint timeoutLimit, uint timeoutMinimum, uint timeoutMaximum)` sets a timeout parameters for a peer. The timeout parameters control how and when a peer will timeout from a failure to acknowledge reliable traffic. Timeout values used in the semi-linear mechanism, where if a reliable packet is not acknowledged within an average round trip time plus a variance tolerance until timeout reaches a set limit. If the timeout is thus at this limit and reliable packets have been sent but not acknowledged within a certain minimum time period, the peer will be disconnected. Alternatively, if reliable packets have been sent but not acknowledged for a certain maximum time period, the peer will be disconnected regardless of the current timeout limit value.

`Peer.Disconnect(uint data)` request a disconnection from a peer.

`Peer.DisconnectNow(uint data)` force an immediate disconnection from a peer.

`Peer.DisconnectLater(uint data)` request a disconnection from a peer, but only after all queued outgoing packets are sent.

`Peer.Reset()` forcefully disconnects a peer. The foreign host represented by the peer is not notified of the disconnection and will timeout on its connection to the local host.

### Classes
#### Host
Contains a managed pointer to the host.

`Host.Dispose()` destroys the host.

`Host.IsSet` returns a state of the managed pointer.

`Host.PeersCount` returns a number of connected peers.

`Host.PacketsSent` returns a total number of packets sent during the session.

`Host.PacketsReceived` returns a total number of packets received during the session.

`Host.BytesSent` returns a total number of bytes sent during the session.

`Host.BytesReceived` returns a total number of bytes received during the session.

`Host.Create(Address? address, int peerLimit, int channelLimit, uint incomingBandwidth, uint outgoingBandwidth, int bufferSize)` creates a host for communicating with peers. The bandwidth parameters determine the window size of a connection which limits the number of reliable packets that may be in transit at any given time. ENet will strategically drop packets on specific sides of a connection between hosts to ensure the host's bandwidth is not overwhelmed. The buffer size parameter is used to set the socket buffer size for sending and receiving datagrams. All the parameters are optional except the address and peer limit in cases where the function is used to create a host which will listen for incoming connections.

`Host.EnableCompression()` enables packet-level compression.

`Host.PreventConnections(bool state)` prevents access to the host for new incoming connections. This function makes the host completely invisible from outside, any peer that attempts to connect to it will be timed out.

`Host.Broadcast(byte channelID, ref Packet packet, Peer[] peers)` queues a packet to be sent to a range of peers or to all peers associated with the host if the optional peers parameter is not used. Any zeroed `Peer` structure in an array will be excluded from the broadcast. Instead of an array, a single `Peer` can be passed to function which will be excluded from the broadcast.

`Host.CheckEvents(out Event @event)` checks for any queued events on the host and dispatches one if available. Returns > 0 if an event was dispatched, 0 if no events are available, < 0 on failure.

`Host.Connect(Address address, int channelLimit, uint data)` initiates a connection to a foreign host. Returns a peer representing the foreign host on success or throws an exception on failure. The peer returned will not have completed the connection until `Host.Service()` notifies of an `EventType.Connect` event. The channel limit and user-supplied data parameters are optional.

`Host.Service(int timeout, out Event @event)` waits for events on the specified host and shuttles packets between the host and its peers. ENet uses a polled event model to notify the user of significant events. ENet hosts are polled for events with this function, where an optional timeout value in milliseconds may be specified to control how long ENet will poll. If a timeout of 0 is specified, this function will return immediately if there are no events to dispatch. Otherwise, it will return 1 if an event was dispatched within the specified timeout. This function should be regularly called to ensure packets are sent and received, otherwise, traffic spikes will occur leading to increased latency. The timeout parameter set to 0 means non-blocking which required for cases where the function is called in a game loop.

`Host.SetBandwidthLimit(uint incomingBandwidth, uint outgoingBandwidth)` adjusts the bandwidth limits of a host in bytes per second.

`Host.SetChannelLimit(int channelLimit)` limits the maximum allowed channels of future incoming connections. 

`Host.Flush()` sends any queued packets on the specified host to its designated peers. 

#### Library
Contains constant fields.

`Library.maxChannelCount` the maximum possible number of channels.

`Library.maxPeers` the maximum possible number of peers.

`Library.maxPacketSize` the maximum size of a packet.

`Library.version` the current compatibility version relative to the native library.

`Library.Initialize(Callbacks inits)` initializes the native library. Callbacks parameter is optional and should be used only with a custom memory allocator. Should be called before starting the work. Returns true on success or false on failure.

`Library.Deinitialize()` deinitializes the native library. Should be called after the work is done.

`Library.Time` returns a current local monotonic time in milliseconds. It never reset while the application remains alive.
