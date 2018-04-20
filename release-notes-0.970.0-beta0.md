# Overview to Ballerina 0.970.0-beta0
This release packages significant enhancements to Ballerina language syntax and type system.

The enhancements include the introduction of object type where you can either define the methods of the object within the definition itself or you can bind them from outside the object.

We have introduced  union type in this release. Union types are types whose set of values is the union of the value spaces of its participant types. Note that with the introduction of union and type switching, we no longer need type casting

The match statement is introduced to help manage union constructs. Unions have to be matched and tuples have to be destructured using the match statements.

A but expression is also introduced facilitate expression evaluation. If the value of the evaluated expression matches to some given type, then we can update the result for a preferred value.

The safe navigation operator has been enhanced to allow safe navigation in the event of null or an error. This will be useful with maps that now have error as common response status.
Nil-lifting navigation has been introduced to help eliminate null pointer exceptions.
Error-lifting navigation has been introduced where you can lift errors when navigating through fields, using the “!” operator.

This release has replaced const with @final for defining constants.

This release has also replaced enums with types, where you can define a new type with a pipe (‘|’) separated list of values.

An error elimination operator has been introduced. You can use the operator if there are errors returned from a function, they will be thrown. You need to use this with caution when you are absolutely sure that an error won’t be returned or the error will be handled by upstream code.

A new concept of nil has been introduced. Note that nil is not the same as null. null should be avoided in general and we have to use nil, which is denoted with , (). null is now only used with JSON type, because, possible set of values for JSON is a union of several types including null.

This release also introduced first class support for streaming event processing. It allows you to build streaming queries with user friendly syntax. The queries include projection, filtering, windows, stream joins ,and patterns.

# Improvements
## Language & Runtime
- Introduction of “object” keyword
- Introduction of type creation capability along with union types
- Introduction of “match”” expression
- Introduction of “but” statement
- Introduction of readonly annotation
- Safe navigation support
- “const” keyword has been replaced with “@final”
- “enum” concept is replaced with types creation
- Multiple value return type function concept is removed and Tuple needs to be used to return multiple value.
- Inline tuple destructuring capability
- Introduction of error elimination operator
- Introduction of Nil type
- Introduction of Streaming capabilities
- Table literal support
- Introduction of transaction commit/failure callbacks
- Introduction of fail statement
- Removal of Transformer syntax


## Standard Library
- Config API
  - Improved API with support for retrieving configs of different data types
  - Support for securing sensitive data
- HTTP
  - OCSP stapling
HTTP caching support
Support for publishing HTTP trace logs to sockets and files
HTTP access logs
Per service chunking configuration
Connection throttling support
Improved APIs for HTTP header related operations
Support for service versioning
Support for HTTP version 2 (HTTP/2)   
Introduce a new method called “setPayload()” to http request and response to take any (string, xml, json, blob, io:ByteChannel or mime:Entity[]) type of payload
- MIME
  - Introduce a new method to get the base type from a given content type
  - Introduce a new method to set any(string, xml, json, blob,io:ByteChannel or mime:Entity[]) type of body to entity
  - Improve entity body operations to rely on content type
  - Improved APIs for MIME specific base64 encoding and decoding
- WebSocket
  - The WebSocket upgrade resource has been moved to the Http Service
  - A new function called acceptWebSocketUpgrade has been introduced to the http:Listener
  - The WebSocket resource signatures have endpoints and not WebSocketConnector as the first argument.
  - The WebSocket resource signatures now have basic data types instead of the frames in previous implementations.
- WebSub
  - Support for WebSub Subscriber, Hub and Publisher
- Observe
  - Support for observability through metrics and tracing APIs
- SQL
  - Support MySQL and H2 packages to interactions with respective DBs.
  - Add mirror table support to read write directly via tables
- Util
  - Improved APIs for base64 encoding and decoding

## IDEs & Language Server
- Language Server
  - Support find all references, Go to definition and Hover provider support for Match expression, Tuple support, Union type support, final and readonly variables, Endpoint, new service syntax, and Object and record statements.
  - Support documentation syntax
  - Support Annotation syntax
  - Support Completion Support for Records, Objects and Endpoints syntaxes
- Composer
  - Introduction of trace log analyzer tool
  - Removal of Transformer UI
  - Support match statement visualization in diagram
  - Support source generation support for match statement, transaction statement

## Ballerina API Gateway
- Annotation support for securing Services
- API Endpoint which is secured by default
- HTTP request interceptor support
- Simple file based userstore
- Basic and JWT based authentication

## Ballerina Observability
- Add out of the box observability through Logging, Metrics and Distributed Tracing

## Ballerina Extensions
- Add ballerina config file support for docker annotations
- Add ballerina config file support for kubernetes annotations


# Getting Started
You can download the Ballerina distributions, try samples, and read the documentation at http://ballerinalang.org. You can also visit [Quick Tour][1] to get started. We encourage you to report issues, improvements, and suggestions at [Ballerina Github Repository][2].

[1]: https://ballerinalang.org/docs/quick-tour/quick-tour
[2]: https://github.com/ballerina-platform/ballerina-lang/
