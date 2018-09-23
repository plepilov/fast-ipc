![HeadBanner](docs/imgs/headbanner.png)
[![Build status](https://ci.appveyor.com/api/projects/status/2x2fegcdfr93hgko/branch/master?svg=true)](https://ci.appveyor.com/project/dvoaviarison/fast-ipc/branch/master)
# Fast IPC
Fast IPC is an open source library that supports typed messages and brings inter-process communication at a higher level for better usability.
It includes:
- Inter process communication layer using named pipes. It supports smart generation of pipe name in case of parent/child processes. Other means of communication are going to be supported in the near future
- Super fast serialization using protobuf
- Typed event driven syntax using internally .Net built-in event capability and exposing simple api such as `Subscribe` and `Publish`

## Get started
To make two processes communicate, all you need is to create an IPC bus in each process, then listen/publish on that bus, as follows:

**Define your messages**: All messages only need to inherit from `Message` and be a `ProtoContract` as follows:
```csharp
[ProtoContract]
public class Ping : Message { }

[ProtoContract]
public class Pong : Message { }
```

**ProcessA**: One process has to be described as `In`
```csharp
static void Main(string[] args) {
	var pipeName = new SimpleStringPipeName(name: "Example");
	var bus = new NamedPipeBus(pipeName: pipeName);
	new ProcessAsHost(bus);
	...
}
```

```csharp
public class ProcessAsHost : IHandleMessage {
	private readonly IBus _bus;

	public ProcessAsHost(IBus bus) {
		_bus = bus;
		_bus.Subscribe(this);
	}

	public void Handle(Message.Message msg) {
		HandleInternal((dynamic)msg);
	}

	private void HandleInternal(Ping msg) {
		Console.WriteLine("Received a ping, sending a pong");
		_bus.Publish(new Pong());
	}

	private void HandleInternal(Pong msg) {
		Console.WriteLine("Received a pong");
	}
}
```

**ProcessB**: The other has to be described as `Out`
```csharp
static void Main(string[] args) {
	var pipeName = new SimpleStringPipeName(
		name: "Example", 
		side: Side.Out);
	var bus = new NamedPipeBus(pipeName: pipeName);
	new ProcessBHost(bus);
	...
}
```

```csharp
public class ProcessBHost : IHandleMessage {
	private readonly IBus _bus;

	public ProcessBHost(IBus bus) {
		_bus = bus;
		_bus.Subscribe(this);
	}

	public void Handle(Message.Message msg) {
		HandleInternal((dynamic)msg);
	}

	private void HandleInternal(Ping msg) {
		Console.WriteLine("Received a ping, sending a pong");
		_bus.Publish(new Pong());
	}

	private void HandleInternal(Pong msg) {
		Console.WriteLine("Received a pong");
	}
}
```

And voila! ProcessA and ProcessB are communicating

Working examples are available in `src` folder.

## Contribute
This project is open source. Fork then PR!

For now we are using named pipes with limitation to 2 processes communicating.
We want to introduce TCP and enable multi-tier IPC.

## Compatibility
- .Net Framework: net40 and up
- .Net Standard: netstandard2.0 and up
