Unity3D OSX + ZeroTier SDK
====

Welcome!

We want your Unity apps to talk *directly* over a flat, secure, no-config virtual network without sending everything into the "cloud". Thus, we introduce the ZeroTier-Unity3D integration!  

Our implementation currently intends to be the bare minimum required to get your Unity application to talk over ZeroTier virtual networks. As a result, we've created an API that is very similar to the classic BSD-style sockets API. With this basic API it is possible to construct more abstracted network layers much like Unity's LLAPI and HLAPI.

***
## API  

- `Join(nwid)`: Joins a ZeroTier virtual network
- `Leave(nwid)`: Leaves a ZeroTier virtual network
- `Socket(family, type, protocol)`: Creates a ZeroTier-administered socket
- `Bind(fd, addr, port)`: Binds to that socket on the address and port given
- `Listen(fd, backlog)`: Puts a socket into a listening state
- `Accept(fd)`: Accepts an incoming connection
- `Connect(fd, addr, port)`: Connects to an endpoint associated with the given `fd` 
- `Write(fd, buf, len)`: Sends data to the endpoint associated with the given `fd`
- `Read(fd, buf, len)`: Receives data from an endpoint associated with the given `fd`
- `CLose(fd)`: Closes a connection with an endpoint

***
## Adding ZeroTier to your Unity app

**Step 1: Create virtual ZeroTier [virtual network](https://my.zerotier.com/)**

**Step 2: Add plugin to Unity project**
 - Create folder `Assets/Plugins`
 - Place `ZeroTierSDK_Unity3D_OSX.bundle` in folder

**Step 3: Include wrapper class source**
 - Drag `ZeroTierNetworkInterface.cs` into your `Assets` folder.

**Step 4: Create and use a `ZeroTierNetworkInterface` object**
 - See examples below for how to use it!

***
## Examples

Start by creating a `ZeroTierNetworkInterface` object and calling 
Calling `ZeroTier.Init()` will start the network service in a separate thread. You can check if the service is running by checking `ZeroTier.IsRunning()`. Then, connecting and sending data to another endpoint would look something like the following:
## Using ZeroTier Sockets API
### Server example
```
public class Example
{
	public ZeroTierNetworkInterface zt;

	public void example_server()
	{
		Thread connectThread = new Thread(() => { 
			// Create ZeroTier-administered socket
			int sock = zt.Socket ((int)AddressFamily.InterNetwork, (int)SocketType.Stream, (int)ProtocolType.Unspecified);
			zt.Bind(sock, "0.0.0.0", 8000);
			zt.Listen(sock, 1);

			// Accept() client connection
			int accept_sock = -1;
			while(accept_res < 0) {
				accept_sock = zt.Accept(sock);
			}

			// Read data from client
			char[] msg = new char[1024];
			int bytes_read = 0;
			while(bytes_read >= 0) { 
				bytes_read = zt.Read(accept_sock, ref msg, 80);
				string msgstr = new string(msg);
				Debug.Log("MSG (" + bytes_read + "):" + msgstr);
			}
		});
		connectThread.IsBackground = true;
		connectThread.Start();
	}
}
```

### Client example
```
public class Example
{
	public ZeroTierNetworkInterface zt;

	public void example_client()
	{
		Thread connectThread = new Thread(() => {	
			// Create ZeroTier-administered socket		
			int sock = zt.Socket ((int)AddressFamily.InterNetwork, (int)SocketType.Stream, (int)ProtocolType.Unspecified);
			zt.Connect (sock, "0.0.0.0",8000);
			zt.Write(sock, "Welcome to the machine!", 24);
		});
		connectThread.IsBackground = true;
		connectThread.Start();
	}
}
```
***
## Design and structure of the ZeroTier Unity OSX Bundle

XCode:
New XCode project
Select Cocoa bundle as target
Add C linkages to external functions
Build as 64bit (not universal)

Unity:
Select x86_64 build target in `Build Settings`
In new C# script asset:

```
[DllImport ("ZeroTierUnity")]
private static extern int unity_start_service ();
```

Add asset to GameObject
Start ZT service

***
## Future Roadmap  
With the ZeroTier sockets API in place, higher-level functionality such as lobbies, chat, and object synchronization could easily be built on top.

