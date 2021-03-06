# ZeroRPC.NET

[![Build status](https://ci.appveyor.com/api/projects/status/i8e6i0iegbrdi0jc?svg=true)](https://ci.appveyor.com/project/denikson/zerorpc-dotnet)

ZeroRPC.NET is a 100% CIL implementation of [zerorpc](http://www.zerorpc.io/), a simple library for communication and Remote Procedure Calls between distributed processes.

Current version of ZeroRPC.NET supports .NET Framework 3.5 and zerorpc version 3.

The API and core logic were mimicked off [zerorpc-node](https://github.com/0rpc/zerorpc-node), but modified to work within .NET.

## Features

* Supports .NET 3.5 and Unity
* Simple API for zeroservices through `IService` interface
* Replaceable paramter/return value (de)serializer
* Synchronous method invoking w/ first-class exceptions
* Asynchronous invoking through callbacks

## Current progress

As of right now, the library is in early beta stage: no official release is available, some useful client-side extensions are missing and not everything has been tested properly yet. However, the library is capable of successfully communicating with zeroservices written in both Python and Node.JS.

## Basic example

### Server

```csharp
using ZeroRpc.Net;
using ZeroRpc.Net.ServiceProviders;

namespace ClientExample 
{
    public class ExampleService
    {
        [MethodDocumentation("Echoes the provided string back")]
        public string Echo(string str)
        {
            return str;
        }
    }

    public class ServerShowcase
    {
        public static void Main(string[] args) 
        {
            // ZeroRPC.NET provides a built-in zeroservice that exposes the methods in a provided object
            SimpleWrapperService<ExampleService> service = new SimpleWrapperService<ExampleService>(new ExampleService());

            Server s = new Server(service);

            // Bind the server to local host port 1234
            // The server will provide its zeroservice through TCP
            s.Bind("tcp://127.0.0.1:1234");
            Console.WriteLine("Now serving at 127.0.0.1:1234!");

            // Most errors are fired asynchronously as to not prevent the main data flow
            s.Error += (s, args) => Console.WriteLine(errorArgs.Info);

            // Prevent the console from closing, since Bind returns immediately
            Console.ReadKey();

            // Dispose of the server in the end.
            s.Dispose();
        }
    }
}
```

### Client

```csharp
using ZeroRpc.Net;

namespace ClientExample 
{
    public class ClientShowcase
    {
        public static void Main(string[] args)
        {
            Client c = new Client();

            // Most errors are fired asynchronously as to not prevent the main data flow
            c.Error += (s, errorArgs) => Console.WriteLine(errorArgs.Info);

            // Connect to a local zeroservice on port 1234 through TCP
            c.Connect("tcp://127.0.0.1:1234");

            // Supports simple synchronous calls with first class exceptions

            try 
            {
                Console.WriteLine(c.Invoke<string>("Echo", "Hello, world!"));
            } catch (RemoteException re) 
            {
                Console.WriteLine($"Error: {re.ErrorName}");
                Console.WriteLine($"Remote stack trace: {re.RemoteStackTrace}");
            }

            // Dispose of the client in the end.
            c.Dispose();
        }
    }
}
```
  
