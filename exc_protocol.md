## <ins>Building a Protocol</ins>
In this exercise we are going to build a simple protocol for communicating 
between a server and a client

A protocol is first and foremost a *contract* - what the other side in 
the communication can expect to find.


### <ins>Message Structure</ins>
Each message will *always* start in a request and end in a response. 
Each request and response will have a similar structure:
```
[Type][Data]
```
where `Type` is a single character indicating which type of request is being made (or which request is being responded to) 
and `Data` includes everything else - the actual message.


### <ins>Data Structure</ins>
The `Data` part itself is built from different *chunks* in the structure:
```
[ChunkLen][ChunkData]
```
Where `ChunkLen` indicates the length in bytes of the specific chunk 
(which is always represented by a two-digit number ranging from `00` to `99`). 
Whenever the `ChunkData` part includes only the characters `00`, it means this is a
`NullChunk` that indicates the end of the message.

### <ins>Control Messages</ins>
Sometimes, an error will occur. What if a request does not comply with the contract we 
agreed on? In those cases, the server will not try to create a 
regular response, and instead will respond in a *control message* - indicating that
something went wrong. the `Type` of control messages will always be `0`, which is 
a protected character that will never be used as the type of regular requests or 
responses.
Aside from the special `Type`, the response looks the same as regular messages - after 
`Type` there will be chunks (each containing both length and the chunk itself) and ends
in a `NullChunk`.

<br>

## <ins>Available Requests</ins>
### <ins>Execute</ins>
The first request the client can make will have the `Type` 1, and will be the 
`Execute` request. It will send a command to the terminal that should be executed, along
with all the necessary arguments for this command.

The request should pass to following data:
``` 
[Type][ArgList]
```
where `ArgList` includes the actual command and all of its arguments - split into chunks
as described above.

The response should also use `Type` 1, and will be built like this:
```
[Type][StdOut][[StdErr][ErrorCode]
```
where everything after the `Type` is part of `Data`. Both `StdOut` and `StdErr` might 
be empty, depending on the command the terminal executed, but `ErrorCode` should always 
be a number between 0-255. Each of `StdOut`, `StdErr`, and `ErrorCode` should be made of
chunks and end with a `NullChunk`
