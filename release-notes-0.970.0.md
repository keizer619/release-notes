# Overview to Ballerina 0.970.0

We proudly announce General Availability of Ballerina 0.970.0. Download now! Ballerina 0.970.0 is an exciting new release of the programming language. 

The 0.970.0 release represents significant changes from the 0.964.0 release and is not backwards compatible. This release represents the changes after a series of design reviews that had material impact on the language design, syntax, and toolchain. The changes are so significant that to those with previous exposure, Ballerina will appear as an entirely new language. 

With these changes, the release notes cover the features of the language from a design point of view, and we are forgoing the traditional approach of release notes covering the incremental changes from release to release. We hope to hit 1.0 milestone before the end of 2018 and will guarantee backwards compatibility at that stage.

Key highlights of Ballerina capabilities included into this release are:

**Concurrent**: Worker support for defining parallel execution units with fork/join semantics, and asynchronous function invocations, which contains improved BVM scheduler functionality.

**Transactional**: First class support for transaction semantics, including distributed transactions.

**Textual and graphical syntaxes**: Sequence diagrams focused on showing worker and endpoint interactions. Endpoints represent network services. Each worker is sequential code, that can zoom in graphically yet represents really textual code. The design is meant to make it easier to understand complex distributed interactions

**Integration specialization**: Brings fundamental concepts, ideas, and tools of distributed system integration into the programming language and offers a type safe, concurrent environment to implement such applications. These include distributed transactions, reliable messaging, stream processing, workflows, and container management platforms. 

Learn more about the [Ballerina philosophy](https://ballerina.io/philosophy/).

# Language

## Values and Types
Ballerina programs operate on a universe of values, and each value belongs to only one basic type such as `int`, `boolean`, `map`, `record`, `function`, etc. There are three kinds of values corresponding to three kinds of basic types. They are simple values (e.g., `int`, `string`, `boolean`), structured values (e.g., `record`, `map`, `array`), and behavioral values (e.g., `function`, `object`).

### Simple Basic Types
The types `int`, `float`, `string`, `boolean`, `blob`, and `nil` are called simple basic types because they are basic types with only simple values. Simple values are always immutable.
* **nil** - The `nil` type has only one value that is denoted by `()`. 
* **boolean** - The `boolean` type has only two values named `true` and `false`. The implicit initial value of a variable of type `boolean` is `false`.
* **int** - The `int` type denotes the set of 64-bit signed integers. The implicit initial value of a variable of type `int`  is `0`.
* **float** -  The `float` type denotes double precision IEEE 754 floating point numbers. The implicit initial value of a variable of type `float` is `+0.0`.
* **string** -  The `string` type denotes the set of sequences of unicode code points. The implicit initial value of a variable of type `string` is the empty sequence.
* **blob** -  The `blob` type denotes the set of sequences of 8-bit bytes . The implicit initial value of a variable of type `blob` is the empty sequence. 

### Structural Basic Types
Structured basic values create structures from other values. A structured value belongs to exactly one of the following basic types:
* **Tuple** - A `tuple` is an immutable list of two or more values of fixed length. 
* **Array** - Arrays are mutable lists of values with dynamic length where each member of the list is specified with the same type. The implicit initial value of an array is the array with a length of `0`.
* **Map** - The `map` type defines a mutable mapping from keys that are strings to values of the same type.
* **Record** - A mutable mapping from keys, which are strings, to values; specifies maps in terms of names of fields (required keys) and value for each field.
* **Table** - The `table` type is a data structure that organizes information in rows and columns. Can be used to create an in-memory table using a type constraint, insert data to it and then access/delete the data.
* **XML** - The `xml` type represents a sequence of zero or more XML items. Each item can be an element, a text, a comment, or a processing instruction.


### Behavioural Basic Types
The following are Ballerina's basic behavioural values.
* **Function** - A `function` with zero or more specified input parameter types and a single return type.
* **Future** - A `future` value represents a value to be returned by an asynchronous function invocation. 
* **Object** - An `object` is a collection of public/private typed fields along with attached functions that allows you to create new, user-defined data types with behavior. The implicit initial value would be an `object` where each field has the implicit initial value for its type.
* **Stream** - The `stream` type represents a stream of events, which allows publishing events to the stream and subscribing to receive events from the  stream. 

### Other Types.
* **Union Type** - The `union` types are types whose set of values is the union of the value spaces of its component types. For example, you can use a variable of a union type to store a `string` or an `int`, but there is only one value at any given time. Syntactically, you can define a union type with component types separated by "|" (vertical bar). 
* **Optional** - One of the design principles of the type system of Ballerina is to eliminate null reference errors. Over the years, null reference errors have caused numerous system crashes, security vulnerabilities, etc. Optional types in Ballerina allows developers to identify where the value or the function is of type `T` optionally, for any T. You can syntactically represent this as `T?` or `T|()`.
* **Any** - The `any` data type is the root of the Ballerina data types. It can represent a variable of any data type. When you do not have prior knowledge of the data type of your variable, you can assign `any` as the type. Values of these variables can come from dynamic content such as the request and response messages and the user input. The `any` type allows you to skip compile-time data type checks.
* **JSON** - JSON is a textual format for representing a collection of values: a simple value (`string`, `number`, `true`, `false`, `null`), an array of values, or an object. Ballerina has a single type named `json` that can represent any JSON value. Thus, `json` is a built-in union type in Ballerina, which can contain values of type `string`, `float`, `boolean`, an `array of any`, or a `map of any`.


## Expressions

### Field access
Field access is the syntax of accessing child elements inside structural typed values, such as objects, records, JSON, XML, etc. Fields can be accessed using two operators:
* Dot operator - Name of the field precedes by a dot, e.g., foo.bar
* Index operator - Name of the field comes within two brackets. Name can be any `string` value expression, e.g., foo[bar]

Both of these operators perform the nil-lifting by default. That is, it allows to walk down the child fields, without worrying whether there will be null along the way. In an event of null, it will stop the navigation and the value of the entire expression will be null.

### Array Access
Array elements can be accessed by the index using the index operator. The index always has to be an integer valued expression. Similar to accessing fields, accessing arrays also performs the nil-lifting default, e.g., foo[index], foo[2].

### Match Expression
Match expression is a way of checking the type of an expression and executing some other expression based on the type of the first expression. It is a form of a type switch. Match expression contains patterns inside, with a type associated to it. Type of each pattern should be matched to at-least one of the types of the expression that is being tested. 

### Elvis Operator
Elvis operator is a conditional operator that can be used to handle `null` values. It evaluates an expression and if the value is `null`, executes the second expression. The elvis operator takes two operands and uses the '?:' symbol to form it.

## Control Flow Statements

### If/Else Statement
An `if/else` statement provides a way to perform conditional execution. It contains three sections: an `if` block, followed by multiple `else if` blocks, and finally a single `else` block. All the `else if` blocks and the `else` blocks are optional. Any number of statements can be defined inside each of these blocks.

### Match Statement
A `match` statement is a type switching construct that allows selective code execution based on the type of the expression that is being tested. The `match` statement can have one or more patterns with a type associated to it. Each pattern have statements that will get executed if that type is matched.

### While
The `while` looping construct will iterate and execute the code block within the while block continuously, until the condition for while loop is `true`. 

### Foreach
The `foreach` looping construct will traverse through the items of a collection of data such as arrays, maps, JSON, and XML and execute the given code block.

With the above two looping constructs, statements such as `break` and `next` can also be used, where the `break` statement would end the loop and `next` statement would go to the next iteration in the loop. 

### Iterable Operations
Iterable operations can be used with types such as `array`, `map`, `json`, `table`, and `xml`. Currently available iterable operations are `map`, `filter`, `count`, `average` and `foreach`. The `map` is a transform operation that can transform the given source element to a target element. The `filter` operation can filter the given element for the given condition. The `count` , `average` and `foreach` are terminal type of operations that will terminate and either give a result out or will consume the elements and terminate the iteration.


## Transaction
A Ballerina transaction is a series of data manipulation statements that must either fully complete or fully fail, and thereby leave the system in a consistent state.  Ballerina supports following three types of transactions over databases and message brokers.

Local Transactions - The transaction performed over single database or a message broker.

XA Transactions - The transaction performed over multiple databases and/or message brokers using the XA two phase commit protocol.
Distributed Transactions - Implements transactional behavior for micro-services by using an external coordinator to execute a two-phase-commit based protocol to manage service resources.

The transaction block is used handle Ballerina transactions. Any transaction aware operations performed within the transaction block are automatically committed or rolled back at the end of the block. Within transaction block, `abort` or `retry` statements can be used to explicitly force to abort the transaction or retry the transaction. 

## Error Handling
Typically, errors indicate erroneous conditions in the program. The Ballerina approach is to introduce a first-class error concept that can both be returned as yet another return value (and thereby processed by the caller as it deems fit) or be thrown.

In the case of an error situation where the caller can recover from it, this error must be returned from the function, rather than throwing the error. Generally, errors will only be thrown, in the case where the caller cannot recover from the error, hence, the program will not be able to continue.

### Try-Catch-Finally
The `try-catch-finally` blocks are basically used for catching and handling exceptions thrown during execution. The try block contains the operations you need to execute. The catch block is used to handle the exceptions that are thrown by the operations in the try block. You can define many catch blocks to catch the different error types that are thrown. The finally block is always executed even if errors are thrown or not. Therefore, you can add all the crucial operations to the finally block.

### Error Lifting
The error lifting operator allows to walk down a set of fields of an object or tuple, without worrying whether there will be a error or null along the way. In an event of error/null, it will stop the navigation and the value of the entire expression will be error/null.

### Check
`check` is a unary expression that is used to handle errors. The `check` expression removes the `error` type from the result of the sub expression and handle error case separately. If `check` is used within a `function`, having `error` as the `return` type or one of the return types, then upon an error, it causes to immediately return from the enclosing function with that error. Otherwise, error will be thrown. With this behavior, check expression can also be considered as a built-in match.

## Concurrency Constructs
The concurrency model is based on workers, which are the fundamental execution units defined in Ballerina. The worker model is designed to be very light-weight constructs, which follows a fully non-blocking approach in its executions, which in-turn makes sure it has optimal utilization of the CPU. 

### Workers
In Ballerina, every callable unit, that is, a function, action or a resource is made up of one or more workers. A worker is a concurrent execution unit, which is independently run when a function call is made. If the user does not define an explicit worker, a default implicit worker is automatically defined, and the function contents are added to it. 

### Asynchronous Functions
Any function or action can be invoked in an asynchronous mode by prefixing the call with the `start` keyword. The return value of this expression will always be a `future` object. The future object can be constrained to a specific type, to be equal to the result type that the function is returning. If not, for void functions, it will be unconstrained. The returned future object can be used to extract the result by blocking until the function call is done, check the current status of the asynchronous call, or cancel the running execution. 

### Fork/Join
The fork/join construct in Ballerina is used in order to split (`fork`) the current function execution to multiple workers, do some processing in parallel, and `join` together the results of the workers to a single execution again. 

### Lock
Ballerina locks are used for concurrency management, encapsulating a block of statements will acquire the locks for each global or service level variable reference used within those block of statements.

## Functions
Ballerina functions operate the same way as any other language. It is a mechanism to create a reusable unit of functionality within a program. Ballerina functions can have three types of parameters.

Required parameters - Set of parameters that are mandatory to be provided, when invoking the function.

Defaultable parameters - They are optional parameters with a default value. Such a function can be invoked without providing a value for those defaultable parameters. When invoking the function, the defalutable parameter needs to be passed as key-value pairs, and their position within the function call does not matter.

Rest parameter - The rest parameter can take zero or more number of values of the given type. Inside the function, the rest parameter is equivalent to an array of the same type.

### Function Pointers
A function pointer is a Ballerina type that allows you to use functions as variables, arguments to functions, and function return values. The name of a function serves as a pointer to that function when called from other functions or operations. The definition of the function name provides the type of the pointer in terms of a function signature.

### Lambda
Lambdas are a syntactic shortcut for defining inline functions. In comparison to a normal function definition, the only missing part is the name.

# Syntax
Ballerina’s underlying language semantics were designed by modeling how independent parties communicate via structured interactions. Subsequently, every Ballerina program can be displayed as a sequence diagram of its flow with endpoints, including synchronous and asynchronous calls. Ballerina’s syntax has both textual and graphical representation designed around sequence diagrams,therefore, the way a developer thinks when writing Ballerina code encourages strong interaction best practices. 

## Textual Syntax
Ballerina’s textual syntax is largely inspired by C, Java, and Go languages. The key language constructs in Ballerina are as follows. 

Function
```ballerina
import ballerina/io;

function add(int a, int b) returns (int) {
    return a + b;
}

function main(string... args) {
    int result = add(5, 6);
    io:println(result);
}
```

Worker
```ballerina
import ballerina/io;
function main(string... args) {
    worker w1 {
        io:println("Hello, World! #w1");
    }    worker w2 {
        io:println("Hello, World! #w2");
    }    worker w3 {
        io:println("Hello, World! #w3");
    }
}
```

Service
```ballerina
import ballerina/http;

service<http:Service> hello bind { port: 9090 } {

    sayHello(endpoint caller, http:Request req) {
        http:Response res = new;
        res.setPayload("Hello, World!");
        var result = caller->respond(res);
    }
}
```

## Graphical Syntax

Ballerina’s graphical syntax resembles a sequence diagram. The control flow within a worker is represented with flow diagram based elements.

Graphical representation of a service with network interactions. 

Ballerina platform comes with the Composer IDE, which allows you to edit and view Ballerina programs graphically and textually. VS Code plugin can be also used to view Ballerina programs graphically.

# Integration Specialization

Ballerina has first class support for services and endpoints. HTTP/HTTP2, WebSockets, WebSub, gRPC, and JMS are some of the available service types. These services are exposed via listener endpoints, which can be secured and monitored. Client endpoints connect to different types of external endpoints and they are inherently resilient. Additionally, commonly used integration message formats, such as XML and JSON, are built-in to the type system of the language. In the context of integration specialization, the following are the released features.

## HTTP
Dispatching to service based on service version by introducing service config annotation to declare versioning rules

```ballerina
@http:ServiceConfig {
    basePath:"/hello/{version}",
    versioning:{
       pattern:"v{Major}.{Minor}",
       allowNoVersion:true,
       matchMajorVersion:true
    }
}
```

Primitive type support for the Path parameter. The primitive types are `string`, `int`, `float`, and `boolean`
```ballerina
@http:ResourceConfig {
     path:"/product/{id}/{name}/{price}"
}

productInfo(endpoint caller, http:Request req, int id, string name, float price) {
     // some code
}
```

- HTTP caching support for client/server endpoints
- HTTP access logs support
- Connection throttling support for client endpoint
```ballerina
endpoint http:Client clientEP {
    url: "some url",
    connectionThrottling: {
        maxActiveConnections: 5,
        waitTime: 30000
    }
};
```

Introduce `setPayload()` to the HTTP request and response to take any type of payload. The `any` type can be `string`, `xml`, `json`, `blob`, `io:ByteChannel`, or `mime:Entity[]`

- Improved APIs for HTTP header related operations
- Chunking support for per service
- Functionalities supported for HTTPS,
  - Certificate validation with CRL, OCSP, OCSP Stapling
  - Configuration for SSL/TLS ciphers and protocols 
  - Hostname verification support
  - Mutual Authentication support
- HTTP2
  - Seamless upgrade from HTTP/1.1 to HTTP/2.0 protocol
  - Server Push support
  - SSL/TLS support with ALPN 

## WebSockets  
WebSocket client/server endpoint supports the following features.

Read/write support for Text, Binary, Ping/Pong, and Close WebSocket frames. The following is a sample code for handling WebSockets Text and Binary frames.
```ballerina
service <http:WebSocketService> wsService bind {port:9090} {
    onText(endpoint ep, string text, boolean finalFrame) {
        ep->pushText(text, final = finalFrame)
                but { error e => log:printError("Error", err=e) };
    }

    onBinary(endpoint ep, blob data, boolean finalFrame) {
        ep->pushBinary(data, final = finalFrame)
                but { error e => log:printError("Error", err=e) };
    }
}
```

- WebSockets secure connections support (`WSS://`)
- Custom headers support for handshake response message
- Sub-protocols negotiation support
- Connection idle timeout support
- WebSocket service endpoint supports query and path parameters for upgrade request and maximum WebSocket frame size configuration.

## Resiliency
Retry support for HTTP client endpoint
Circuit breaker support for the HTTP client endpoint to gracefully handle network failures. Following is a sample of CircuitBreaker configuration:

```ballerina
endpoint http:Client cbClientEP {
   url: "http://localhost:8080",
   // Circuit breaker options
   circuitBreaker: {
       rollingWindow: {
           timeWindowMillis: 10000,
           bucketSizeMillis: 2000
       },
       failureThreshold: 0.2,
       resetTimeMillis: 10000,
       statusCodes: [400, 404, 500]
   },
   timeoutMillis: 2000
};
```

- Failover support for HTTP client endpoint
- Load balancing support for the HTTP client endpoint. Following is the simplified use of a sample of LoadBalanceClient configuration:

```ballerina
endpoint http:LoadBalanceClient lbBackendEP {
   // Define the set of HTTP clients that need to be load balanced.
   targets: [
       { url: "http://localhost:8080/mock1" },
       { url: "http://localhost:8080/mock2" },
       { url: "http://localhost:8080/mock3" }
   ],
   // The algorithm used for load balancing.
   algorithm: http:ROUND_ROBIN,
   timeoutMillis: 5000
};
```

## MIME 
- Ballerina provides  built-in implementation of the MIME specification. Following are some of the features:
- Support discrete media types
- Support composite media types 
  - multipart/form-data
  - multipart/mixed
  - multipart/alternative
  - multipart/relative
- Ability to recognize and separate parts of unrecognized subtypes of multipart entities
- Encapsulate multiple body parts in a single message
- Decode multipart messages

## WebSub
Implementation of the WebSub recommendation that facilitates push-based content delivery/notification mechanism between publishers and subscribers.
Allow introducing WebSub Hubs, Subscribers and Publishers and support WebSub features including subscription and unsubscription, honouring lease seconds values and authenticated content distribution.

Ballerina’s WebSub implementation supports JSON content and provides the following functionality:
Ballerina WebSub Subscriber
A service type that allows two resources - `onIntentVerification` accepting intent verification requests and `onNotification` accepting content delivery requests
Features include:
Sending subscription requests to a hub for a particular topic at start up, where the hub and topic are specified as or derived from an annotation
Auto Intent Verification - if the `onIntentVerification` resource is not specified
Signature validation for authenticated content
Subscription and unsubscription requests could be sent explicitly via Client Endpoints

```ballerina
import ballerina/log;
import ballerina/websub;

endpoint websub:Listener websubEP {
   port: 8181
};

@websub:SubscriberServiceConfig {
   path: "/websub",
   subscribeOnStartUp: true,
   topic: "<TOPIC_URL>",
   hub: "<HUB_URL>",
   leaseSeconds: 3600,
   secret: "<SECRET>"
}
service websubSubscriber bind websubEP {

   onNotification(websub:Notification notification) {
       log:printInfo("WebSub Notification Received: " + notification.payload.toString());
   }

}
```
Ballerina WebSub Hub 
A Hub service which accepts subscription requests from subscribers, and delivers content to the subscribers on notification from publishers
Features include:
Authenticated content distribution
Honouring lease periods

Ballerina WebSub Publisher
Publishers can bring up their own Ballerina Hub, to which they will publish updates, and allow subscribers to subscribe for their topics
Client endpoints are available for publishers to publish updates to remote Hubs
Utility functions are available to add a Link Header specifying hub and self URLs to facilitate WebSub discovery

## gRPC
gRPC is a protocol which is layered over HTTP/2 and enables client and server communication by combination of any supported languages. In gRPC a client application can directly call methods on a server application on a different machine, making it easier for you to create distributed applications and services.

Ballerina also supports creating gRPC client and server as other supported language.  Ballerina based gRPC server/client supports interacting with server and client written either in Ballerina or any other supported languages. For example: you can easily create gRPC server in Ballerina and connect with client in Go. Syntax for writing simple gRPC unary server is like below,

```ballerina
endpoint grpc:Listener listener {
    port:9090,
};

service SamplegRPCService bind listener {
   receiveMessage (endpoint caller, string name) {
       // service resource logic goes here.
       error? err = caller->send(<response message>);
       _ = caller->complete();
   }
}
```

Ballerina follows code first approach to create gRPC service which means you write Ballerina gRPC service like above and Ballerina compiler will generate service contract file(.proto file) automatically. From the service contract, you can either generate client code in Ballerina and from any other supported languages. In Ballerina, the client code are generated using the built-in proto compiler.

The main features of Ballerina gRPC are,
- Ballerina gRPC service supports all RPC types.
- Unary
- Server streaming
- Client streaming
- Bidirectional Streaming
- Ballerina gRPC service supports communicating with client written from any supported language.
- Supports passing/retrieving headers.
- Supports generating client code using the built-in proto compiler.
- Supports error handling.
- Supports secured communication with client and server.

## Database client endpoints
Ballerina Database client endpoints allow you to connect to SQL-based relational database systems and perform data definition, data access, and data manipulation operations on the database. The following database client endpoints are supported: 
- JDBC - For any JDBC supported database.
- MySQL - For MySQL database
- H2 - For H2 database

The following features are supported in all the data clients.
- Data manipulation/definition support via `update`, `call`, `updateWithGeneratedKeys` operations.
- Batch data update support via `batchUpdate` operation.
- Data access support via `select` operation, which allows easy access of data using returned tables.
- Proxy table support, which allows read/delete on the actual DB table via a proxied table.
- All operations are transaction aware. 

```ballerina
endpoint jdbc:Client testDB {
	url: "jdbc:mysql://localhost:3306/testdb",
	username: "root",
	password: "root",
	poolOptions: { maximumPoolSize: 5 },
	dbOptions: { useSSL: false }
};

table<Student> tb = check testDB->select("SELECT * FROM student", Student);
```

## Messaging endpoints
Messaging connectors enable the services and programs written in Ballerina to communicate with messaging backends like Ballerina Message Broker and ActiveMQ. The messaging connector API provides features like transactions, message headers, properties, and different acknowledgement mode support to address the common requirements of a modern integration engineer when integrating with a messaging system.

The Ballerina JMS API provides the constructs to connect to any JMS provider as shown below.

```ballerina
endpoint jms:SimpleQueueReceiver consumer {
   initialContextFactory: "bmbInitialContextFactory",
   providerUrl: "amqp://admin:admin@ballerina/default?brokerlist='tcp://localhost:5672'",
   queueName: "MyQueue"
};

service<jms:Consumer> jmsListener bind consumer {
   onMessage(endpoint consumer, jms:Message message) {
      // Application logic
   }
}
```

The Ballerina MB API provides the constructs to connect to the Ballerina Message Broker provided in the Ballerina platform, as shown below.

```ballerina
endpoint mb:SimpleQueueReceiver receiver {
   host: "localhost",
   port: 5672,
   queueName: "MyQueue"
};

service<mb:Consumer> mbListener bind receiver {
   onMessage(endpoint consumer, mb:Message message) {
      // Application logic
   }
}
```

# Standard Library 

The Ballerina standard library provides a set of commonly used functionalities. The following packages are available as a part of the standard library:

## ballerina/auth 
Provides an interface for looking up user data for authentication and authorization purposes. Also it contains a sample implementation that uses a Ballerina configuration file as user registry.

```ballerina
// Check if the user exists (for authentication)
auth:ConfigAuthProvider authProvider;
boolean isAuthenticated = authProvider.authenticate(<username>, <password>);
// Retrieve the scopes of the user (for authorization)
string[] scopes = authProvider.getScopes(<username>);
```

## ballerina/cache 
Provides a configurable in-memory caching solution that supports both time-based eviction and size-based eviction.

## ballerina/config
Provides a configuration lookup and resolve mechanism, with the option for securing configurations, and an API for reading these configurations.

```ballerina
// Look up a string config value
string host = config:getAsString(“http.host”);
// Look up an int config value. If the specified key cannot be
// found, the default value is returned
int port = config:getAsInt(“http.port”, default = 9090);
```

## ballerina/crypto 
Provides a set of functions for some of the commonly used hashing algorithms.

## ballerina/file
Provides a directory listener that can be used to listen to directory events.

```ballerina
import ballerina/file;
import ballerina/log;

endpoint file:Listener localFolder {
    path:"/home/ballerina/Input",
    recursive:false
};

service watcher bind localFolder {
    onCreate (file:FileEvent m) {
	   string msg = "Create: " + m.name;
        log:printInfo(msg);
    }
    onDelete (file:FileEvent m) {
	   string msg = "Delete: " + m.name;
        log:printInfo(msg);
    }
    onModify (file:FileEvent m) {
	   string msg = "Modify: " + m.name;
        log:printInfo(msg);
    }        
}
```

## ballerina/io 
Provides an asynchronous I/O framework to source/sink that reads/writes as bytes, characters, and records.
Reading and writing bytes:

```ballerina
// Retrieving a ByteChannel to the file.
io:ByteChannel channel = io:openFile(filePath, permission);
// Reading the bytes from the channel
var result = channel.read(numberOfBytes)
// Writing some bytes to the channel.
var result = channel.write(content, startOffset);
```

Reading and writing characters:
```ballerina
// Reading/writing characters of different encodings e.g.: utf-8
// First, get the ByteChannel representation of the file.
io:ByteChannel channel = io:openFile(filePath, permission);
io:CharacterChannel characterChannel = new(channel, encoding);
// Read content as characters 
var content = characterChannel.read(numberOfChars);
// Write characters  
channel.write(content, startOffset);

// Reading a JSON
match characterChannel.readJson(){
   json result =>{
       return result;
   }
   error err =>{
      //handle error
   }
}

// Writing a JSON
match characterChannel.writeJson(content){
    //handle match conditions 
}
```

Reading and writing text records:
```ballerina
// Reading and writing delimited text records

io:ByteChannel channel = io:openFile(filePath, permission);
// Create a `character channel` from the `byte channel` to read content as text.
io:CharacterChannel characterChannel = new(channel, encoding);
// Convert the `character channel` to a `record channel`
//to read the content as records.
io:DelimitedTextRecordChannel delimitedRecordChannel = new(characterChannel, rs=rs, fs=fs);
//Read Text Record 
var recordResp = delimitedRecordChannel.getNext();
//Write string [] as records 
delimitedRecordChannel.write(records);
```

Processing CSV records:
```ballerina
io:CSVChannel srcCsvChannel = io:openCsvFile("./filepath");

match channel.getNext() {
string[] fields => {
       	// Process fields 
}
// Handle error and the rest
}
// Write an string [] as CSV
dstChannel.write(records);
```

## ballerina/log
Provides an API for logging.

## ballerina/sql
Provides the common types and constants used across all the database client endpoints. 

## ballerina/math
Provides a set of functions for performing commonly used mathematical calculations and operations.

## ballerina/reflect
Provides utility functions for reading annotations and deep equality checks.

## ballerina/runtime
Provides utility functions and records related to the runtime.

## ballerina/system 
Provides a set of functions for retrieving details related to the system.

## ballerina/task
Provides an API for managing task timers and task appointments.

## ballerina/time
Provides a set of functions for handling, parsing and formatting date and time.

# IDEs & Language Server

## Language Server
Ballerina Language Server provides the code intelligence for Ballerina programming. Ballerina Language Server can be integrated with any Language Server Protocol (LSP) supported development tool to provide consistent code intelligence throughout. Ballerina Language Server comes with the following set of features.

### Code Diagnostics
Identifies and highlights both semantic and syntactic errors.

### Code Completion and Suggestions
Code completion triggers while editing text, and a user can also explicitly trigger completions for a given context.

### Hover
When hovering over an item, an overview of that item appears. For example, if the item is a function, hover support provides an overview of the function signature.

### Signature Help
Signature Help provides a compressed set of function signature information. Signature Help triggers when you type “(“ or “,”. Currently supports only the functions, and on the trigger, user will be shown the function parameters and types, along with the parameter descriptions.

### Goto Definition
Goto Definition will be used in order to jump to the definition of a certain item. Goto Definition can be triggered based on the development tool’s defined trigger options.

### Find References
Finds all the references of a defined item. This supports finding item references in the same file and also references in other packages in the same project.

### Code Action
Code Action prompts the user with a defined set of actions available for a given language construct and also for certain diagnostic types. Code Actions include the following:
- Add imports
- Add missing imports.

### Add Documentation for Top Level Constructs
Supports adding documentation for a single top-level item or all the top-level constructs in a file.

## VSCode Plugin
The Ballerina VSCode plugin includes the following features:
- Syntax highlighting
- Intellisense for Ballerina language via Ballerina Language Server
- Diagramming (view Ballerina programs graphically)
- Debugging

## IntelliJ IDEA
The Ballerina IDEA plugin includes the following features:
- Syntax highlighting
- Code completion and suggestions
- Code formatting
- Go to definitions
- Find usages
- Code diagnostics
- Ballerina program running and debugging

## Composer
Composer is an IDE included with the Ballerina platform that allows you to design and write Ballerina programs textually as well as graphically. It also comes with a set of features targeted for integration development. 
- Graphical interaction flow designing
- Textual editing support for Ballerina 
- Intelligent code completion via Ballerina Language Server
- Run and Debug support for Ballerina programs
- Design-first API development with Open API Specification
- Try-it client
- Dev time service tracing

# Ballerina API Gateway
The Ballerina API Gateway allows users to expose services in a managed manner. The built-in API endpoint in Ballerina supports securing the exposed services. The main features of the API endpoint are:
- Basic authentication
- JWT-based authentication 
- Configuration-file-based authentication provider to store usernames, passwords, scopes, and their associations
- Authorization using scopes
- Specifying authentication and authorization rules at service or resource levels
- Inheriting authentication/authorization rules from service level and overriding at the resource level
- Authentication and authorization at downstream services via JWT token propagation

Sample service secured with basic authentication:
```ballerina
import ballerina/http;

// The endpoint used here is 'http:SecureListener', which by default tries to
// authenticate and authorize each request. The developer has the option to
// override the authentication and authorization at service and resource level.
endpoint http:SecureListener ep {
   port: 9090
};

@http:ServiceConfig {
   basePath: "/hello",
   authConfig: {
      authentication: { enabled: true },
      scopes: ["scope1"]
   }
}
// Auth configuration comprises of two parts - authentication & authorization.
// Authentication can be enabled by setting 'authentication:{enabled:true}'
// annotation attribute.
// Authorization is based on scopes, where a scope maps to one or more groups.
// For a user to access a resource, the user should be in the same groups as
// the scope.
// To specify one or more scope of a resource, the annotation attribute
// 'scopes' can be used.
service<http:Service> echo bind ep {
   @http:ResourceConfig {
      methods: ["GET"],
      path: "/sayHello",
      authConfig: {
         scopes: ["scope2"]
      }
   }
   // The authentication and authorization settings can be overridden at
   // resource level.
   // The hello resource would inherit the authentication:{enabled:true} flag
   // from the service level, and override scope defined in service level
   // (scope1) with scope2.
   hello(endpoint caller, http:Request req) {
      http:Response res = new;
      res.setPayload("Hello, World!!!");
      _ = caller->respond(res);
   }
}
```

# Ballerina Message Broker
Ballerina Message Broker (BMB) is a lightweight, easy to use message broker designed for use with microservices for their messaging requirements. The main features of BMB are:
- AMQP 0-9-1 transport with TLS support
- Active-Passive HA support
- CLI tool to manage the broker
- RDBMS-based message persistence
- Admin REST API
- Messaging metrics for monitoring MB
- Message tracing
- Local and distributed transaction support
- JMS support
- Cloud native config support
- Authentication and authorization extensions
- In-memory mode

# Ballerina Observability
Monitoring, tracing, and logging are three ways of observing a system. Ballerina programs have default instrumentation for metrics monitoring and distributed tracing with [OpenTracing](http://opentracing.io/) compliance. Ballerina metrics monitoring is powered by Prometheus, and it should be configured externally to scrape metrics data from the Ballerina metrics endpoint that is started with each Ballerina VM. Ballerina’s default distributed tracing is powered by Jaeger. Ballerina logs can be read and fed to any external log monitoring system like Elastic Stack to perform log monitoring and analysis.

Ballerina observability enables developers to understand the execution and performance impact introduced by the Ballerina program. For example, by enabling the metrics monitoring and distributed tracing, developers can identify slow services and can drill down from the services to the actual request hop that is causing the delay in the overall request flow. Similarly, by log monitoring and analysis, the additional debug information regarding any unfavorable situations can be revealed to pinpoint the root cause. 

Developers can enable a Ballerina program to collect the data to observe by simply using the `--observe` flag (with default configurations) or passing specific Ballerina configurations when running the Ballerina program. The external systems, such as Prometheus and Jaeger, need to be used to analyze and graphically represent the collected data from a Ballerina program. 

# Ballerina Extensions
Ballerina builder extensions run after the compilation phase. These extensions analyze code to generate deployment artifacts and provide utilities to make deployment of your apps and services easier. 

When a developer starts building a project, the system starts by parsing the source code, which is followed by dependency analysis, compilation, and a phase at which deployment artifact generation can take place. 

These deployment artifacts can be in the form of simple files or complex types like container images, virtual images, etc. The Ballerina builder extension supports the following deployment artifacts:
- Dockerfiles 
- Docker images 
- Kubernetes artifacts

It is possible for third parties and the ecosystem to create their own annotations and builder extensions that generate different kinds of deployment artifacts. You can publish these extensions on Ballerina Central for others to use.

# Tools 
The `ballerina` command is a tool for compiling, running, and building Ballerina programs. It also comes with a set of utilities to manage Ballerina packages and projects.

## Compile and Run a Ballerina Program 
The `ballerina run` command compiles and runs Ballerina program source files, binaries, or packages. If a Ballerina source file (`.bal` file) or a source package is given, the run command compiles and runs it. The compilation is done internally and does not generate a binary file. 
You may use the Ballerina `build` command to compile a source and provide the generated binary file (`.balx` file) to the `run` command. The binary runs much faster than a source file, because the source file run requires a compilation phase.

You must have either a `main()` function or services or both in order to run a program or a package. When both the `main()` function and services are present, `run` executes services first and then executes the `main()` function.

## Init
The `ballerina init` command creates a Ballerina project and some sample source files to get you started. It can additionally create packages and the `ballerina.toml` file when run in interactive mode using `-i` flag. The `ballerina.toml` file is used to specify the organization name and version of the project, which is mandatory when pushing to Ballerina Central.

## Build
The `ballerina build` command compiles Ballerina sources and writes the output to a file. By default, the output filename for a package is the package name suffixed with `.balx`. The default output for a source will have the `.bal` suffix replaced with `.balx`. If the output file is specified with the `-o` flag, the output will be written to the given output file name. 

## Push, Install, and Pull
The `ballerina push` command uploads the given package to the [Ballerina Central](https://central.ballerina.io/) repository. You will be redirected to create an account during the first attempt.

When you push a package to Ballerina Central, it will validate the organization for the user against the org-name defined in your package’s `ballerina.toml` file. Therefore, you must pick the organization name that you intend to push the package into and set that as the org-name in the package `ballerina.toml` file. 

You may also push packages to your home repository if you want to share a package between two projects in the same dev machine using 'ballerina push --repository home'.  'ballerina install' is an alias for the same.

The `ballerina pull` command downloads a package from Ballerina Central to your home repository cache. This same behavior happens when you build a project with a package import. The `pull` command will download it beforehand, so it will be available for build even when offline.

## List
The `ballerina list` command lists dependencies of a program source or package in a tree format.

## Search
The `ballerina search` command searches Ballerina Central for the given text appearing in the organization name, package name, or package descriptions. 

## Test
The `ballerina test` command allows you to compile and run Ballerina test sources and prints a summary of the test results. You can execute tests in a single test file, a test package, or an entire Ballerina project. Use `ballerina test` without any parameters to execute tests for the entire project. Use `ballerina test <packagename>` to execute tests within the specified package. Use `ballerina test <balfile>` to execute the given Ballerina test file. Note that imports from other packages will not work for file executions. File path can be a relative or absolute.

## Doc
The `ballerina doc` command generates API documentation for a given program or package. If no source or package name is given, it will generate API documentation for all the packages in the current project folder. The default output location of API docs is a folder named `api-docs`. 

## Swagger 
The `ballerina swagger` command provides the following Swagger utilities:
- The Swagger to Ballerina utility will generate Ballerina source code for a provided Swagger/OAS3 definition. Ballerina supports two types of code generation. `mock` generation will generate a mock Ballerina service. `client` generation will generate a Ballerina client stub. 
- The Ballerina to Swagger utility can be used to export an OAS3 definition of a Ballerina service. 
Build time client stub generation is also possible with the `@swagger:ClientEndpoint`  and `@swagger:ClientConfig` annotations.

## gRPC
The gRPC protobuf tool is used to generate the sample client and client stub using the service protobuf definition via the 3.4.0 protoc compiler.

## encrypt
The `ballerina encrypt` command encrypts values containing sensitive information such as passwords and secrets. These encrypted values can be used by placing them in a configuration file or by passing them as runtime arguments, which can then be read through the Config API. It will automatically decrypt the encrypted configuration values on demand.

# Runtime
Ballerina is a compiled language. The Ballerina compiler transforms the source code to Ballerina bytecode, which is the instruction set that the Ballerina bytecode interpreter understands. It is also the intermediate representation to which a source program is transformed by the Ballerina compiler.

## Ballerina Virtual Machine (BVM)
The Ballerina Virtual Machine (BVM) is a software process that executes Ballerina programs. BVM is a combination of all of the following components:
- Instruction set (Ballerina bytecode)
- Bytecode interpreter: a virtual CPU that performs the instruction cycle  fetch-decode-execute
- Storage for instructions and operands
- Function call stack
- Instruction pointer, which points to the next instruction to be executed.

The BVM architecture and instruction set are designed based on register-based virtual machine architecture.

### Bytecode interpreter
The bytecode interpreter in BVM is responsible for executing instructions. Instruction is an encoded behavior. It contains an opcode and one or more operands. Here is a sample instruction:

```
iadd 0 1 2 - Add integers in 0th and 1st registers and store the result in the 2nd register. 
```

### Worker Scheduler 
One of the core functionality of the Ballerina VM is the worker scheduler. This is used to schedule all the workers that are executed in the system. During a function invocation, all the workers that belong to the functions are scheduled to be executed in the BVM scheduler. The scheduler will check for available resources and will schedule the workers appropriately. Each worker has a life cycle state that it goes through, which represents being ready, running, waiting for response, waiting on locks, and finally the finished state. The worker scheduling follows a fully non-blocking mode, where the execution threads of the workers will never block for I/O, locks, sleep, etc. Instead, the execution thread will always be freed up for CPU usages for other workers. In this manner, Ballerina ensures the most efficient usage of CPU resources. 

# Getting Started
You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour](https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).
