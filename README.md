#modular_device_protocol

Authors:

    Peter Polidoro <polidorop@janelia.hhmi.org>

License:

    BSD

Modular device clients and servers communicate with one another using
a slightly modified form of [JSON-RPC](http://www.jsonrpc.org/) 2.0, a
light weight remote procedure call protocol using
[JSON](http://www.json.org/) strings.

JSON-RPC is a stateless, light-weight remote procedure call (RPC)
protocol. Primarily this specification defines several data structures
and the rules around their processing.

In JSON-RPC, the Client is defined as the origin of Request objects
and the handler of Response objects. The Server is defined as the
origin of Response objects and the handler of Request objects.

##Request

Clients send servers requests to invoke server methods.

JSON-RPC 2.0 requests are JSON objects that contain four members,
"jsonrpc" specifying the version of the protocol, "method" containing
the name of the method to be invoked, "params" containing the
parameter values, and "id" containing some value used to match a
response to a request.

Example JSON-RPC 2.0 request:

```json
{"jsonrpc": "2.0", "method": "subtract", "params": [42, 23], "id": 1}
```

The modular device protocol uses a compact version of JSON-RPC
requests. Instead of being a JSON object, the compact request instead
uses a JSON array. The "jsonrpc" and "id" members are not used, only
the "method" and "params" members. JSON arrays are ordered collections
of values. The first value in the compact request is the method and
the optional following values are the parameters to the method, if it
has any.

Example compact JSON-RPC request:

```json
["subtract",42,23]
```

The compact request requires fewer server resources to parse than the
full object request making it easier to implement on small embedded
devices with limited processors. The compact request is communicated
more quickly from client to server than the full request and the
compact request also makes it easier to send the server requests
manually from a command line.

Modular device servers use a JSON
[sanitizer](https://github.com/janelia-arduino/JsonSanitizer.git) to
sanitize erroneous, but interpretable JSON strings before parsing the
requests. This allows a super compact form of the request that is very
easy to type manually on a command line when interacting with a
modular device server. The super compact form should ONLY be sent from
command lines since the server response to the super compact form may
mess up the parsers on other modular device clients.

Example of a super compact JSON-RPC request:

```shell
subtract 42 23
```

JSON requires that the highest level root structure be either an
object or an array. The super compact request is not valid JSON, but
it is obviously an ordered set of values, so it can easily be
sanitized into valid JSON by adding the missing outer brackets, by
quoting the unquoted strings, and converting the spaces to commas.

##Response

JSON-RPC 2.0 responses are JSON objects that contain four members,
"jsonrpc" specifying the version of the protocol, "result", "error",
and "id". The "id" value must be the same as the "id" value of the
request. The "result" value is determined by the method that was
invoked on the server and may be null. The "result" member is required
if the request is successful, but must not exist if there was an error
handling the request. The "error" member does not exist in the
response unless there was an error handling the request.

Example JSON-RPC 2.0 response:

```json
{"jsonrpc": "2.0", "result": 19, "id": 1}
```

The modular device response to a compact JSON-RPC request is almost
identical to a JSON-RPC 2.0 response, except the "jsonrpc" member is
missing and the "id" value is the same as the "method" value in the
request.

Example compact JSON-RPC response:

```json
{"id":"subtract","result":19}
```

The server assumes that super compact requests originate from client
commmand lines, so it pretty prints the JSON to make it easier to read
on a terminal. In fact the super compact should ONLY be sent from
command lines since the extra newlines in the server response to the
super compact form may mess up the parsers on other modular device
clients.

Example super compact JSON-RPC response:

```json
{
  "id": "subtract",
  "result": 19
}
```

##Examples

Here are some more request and response examples using the syntax:

```shell
--> data sent to Server
<-- data sent to Client
```

Example JSON-RPC 2.0 request and response with a non-existent method
error:

```json
--> {"jsonrpc": "2.0", "method": "foobar", "id": "1"}
<-- {"jsonrpc": "2.0", "error": {"code": -32601, "message": "Method not found"}, "id": "1"}
```

Example compact JSON-RPC request and response with a non-existent
method error:

```json
--> ["foobar"]
<-- {"id":"foobar","error":{"message":"Method not found","code":-32601}}
```

Example of a successful compact JSON-RPC request and response:

```json
--> ["blinkLed",0.5,0.5,10]
<-- {"id":"blinkLed","result":null}
```

Example super compact JSON-RPC request and response with a
non-existent method error:

```json
--> foobar
<--
{
  "id": "foobar",
  "error": {
    "message": "Method not found",
    "code": -32601
  }
}
```

Reponses to super compact JSON-RPC are pretty printed with additional
newline characters to make it easier to read the reponse in a serial
terminal.

Example of a successful super compact JSON-RPC request and response:

```json
--> getLedPin
<--
{
  "id":"getLedPin",
  "result":13
}
```

