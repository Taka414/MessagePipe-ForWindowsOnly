# MessagePipe-ForWindowsOnly
See below for a description of the original MessagePipe.

https://github.com/Cysharp/MessagePipe



## What WindowsOnly

Although very silly, MessagePipe can be used for inter-process communication(IPC) on Windows as follows.



The following package is being added to the original `MessagePipe.Interprocess` due to its necessity.

>  System.IO.Pipes.AccessControl



Examples of Use

```cs
// Setting PipeSecurity
PipeSecurity pipeSecurity = new PipeSecurity();

// Add read/write permissions to Everyone
pipeSecurity.AddAccessRule(new PipeAccessRule(
    new SecurityIdentifier(WellKnownSidType.WorldSid, null),
    PipeAccessRights.ReadWrite,
    AccessControlType.Allow));

// Add full access rights to current user
pipeSecurity.AddAccessRule(new PipeAccessRule(
    WindowsIdentity.GetCurrent().User,
    PipeAccessRights.FullControl,
    AccessControlType.Allow));

// - - - - - -

// Prepare message pipe
var sc = new ServiceCollection();
sc.AddMessagePipe()
    .AddNamedPipeInterprocess("pipeName", x =>
    {
        x.HostAsServer = true;
        x.PipeSecurity = pipeSecurity; // !!Set Object
        x.UnhandledErrorHandler = (msg, e) => helper.WriteLine(msg + e);
    });
var provider = sc.BuildServiceProvider();

var p1 = provider.GetRequiredService<IDistributedPublisher<int, int>>();
var s1 = provider.GetRequiredService<IDistributedSubscriber<int, int>>();

var result = new List<int>();
await s1.SubscribeAsync(1, x =>
{
    result.Add(x);
});

var result2 = new List<int>();
await s1.SubscribeAsync(4, x =>
{
    result2.Add(x);
});
```



