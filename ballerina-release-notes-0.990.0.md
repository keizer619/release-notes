# Overview to Ballerina 0.990.0

Ballerina 0.990.0 consists of significant improvements of the language syntax, which is based on a new specification.

# Highlights

[Key highlights of Ballerina language included into this release]

# Breaking Language Changes

- Implicit variable initialization has been removed. Therefore the code `int a; io:println(a);` that was valid in the previous release will fail with an error: `variable 'a' is not initialized`.

- Variables must be initialized explicitly before using them. For more information, refer to variable initialization in the [‘What’s new’ section].  

- The `match` statement no longer selects a `block` statement or an expression to execute based on which pattern a type matches. Now it selects a `block` statement based on the patterns a value matches. For more information, refer to the [match statement section]. 

- The `but` expression has been removed. You can use type tests instead. For more information, refer to the [structural types section]. 

- The error type is no longer a built-in record type. Therefore, you will see syntax errors for error literals in the form of `{message: “error message goes here”, cause: e}`. From this release onwards, the error type is a structured basic type used only for representing errors. It contains a reason; a string identifier for the category of error, a detail; a frozen mapping providing additional information, and a stack trace. For more information, refer to the [error handling section]. 

- The `any` type no longer includes the `error` type. The `any` type is now a union of all types except `error` type (and its subtypes). This change forces errors to be documented explicitly, even if a function returns any.

- The `map` type without type parameter is not allowed from this release onwards. The same applies to `future` and `stream` types. For more information refer to the [structural types section].

- The `try-catch-finally` and `throw` statements have been removed from this release. It encourages exception handling mechanism available in languages such as Java, JavaScript, and C++ where they allow mixing both normal errors (which programmers must be aware of and handle) and abnormal errors (which cannot be dealt with and often indicates a program error). For more information, refer to the [error handling section].

- The `check` expression semantics has been changed to not throw an error if `return` type of the enclosing function or resource does not contain the `error` type.

- The `self` keyword has been mandated to access object members within the object definition. The following code will not compile with this Ballerina version.

```ballerina
type Person object {
    private string name;
    function getName () returns string {
        return name;
    }
};

```

Now you need to use `self` when referring to object members as follows. 

```ballerina
type Person object {
    private string name;
    function getName () returns string {
    return self.name;
    }
};

```

Object constructor syntax has been changed from this release onwards. Here is the old syntax.  

```ballerina
type Person object {
    private string name;
    new (name) {
    }
};

```

Here is the new syntax. With this change, we have removed the special signature in the constructor that existed in the previous release. 

```ballerina
type Person object {
     private string name;
     function __init (string name) {
          self.name = name;
    	  }
};

```

For more information, refer to the [object constructor redesign section]. 

- An object method can be defined outside of the object definition given that it is not an abstract object and the object method is declared inside the object definition. The qualified name of the method defined outside is composed of the object type name and the method name. The previous syntax was `object-type-name::method-name` and the new syntax is `object-type-name.method-name`.

- Endpoints and services are the key abstractions in Ballerina that bring network programming to a higher level of abstraction when compared to traditional languages. There are two kinds of endpoints: listener endpoints and client endpoints.  We have changed the listener (inbound) endpoint variable definition syntax. Here is the old syntax.

```ballerina
endpoint http:Listener httpEp {
    port: 9095
};

```

Now listener endpoints are Ballerina objects that implement the abstract listener object. Therefore, you can create a new instance of a module-level listener endpoint as follows. For more information, refer to endpoints and services section. 

```ballerina
listener http:Listener httpEp = new (9095);

```

The service and resource definition syntax has been changed. Here is the old syntax. 

```ballerina
service hello bind httpEp {
    hi (endpoint caller, http:Request req) {
        _ = caller->respond("Hello, World!");
    }
}

```

Now services are first-class values and are like singleton objects. Resource definition has been modified slightly. Now a resource definition looks like a function definition with the resource qualifier. For more information, refer to standard library syntax section.

```ballerina
service hello on httpEp {
    resource function hi (http:Caller caller, http:Request req) {
        _ = caller->respond("Hello, World!");
    }
}

```

- The syntax of defining client endpoints (outbound endpoints) has been simplified. For more information, refer to the [standard library syntax section].

- The annotations `@final` and `@readonly` have been removed from this release onwards. Now you can declare final variables using the `final` keyword. For example, `final int port = readPortFromConfig();`.

- The `@doc` annotation that was deprecated in previous releases has been removed from this release onwards.

- The syntax and semantics of functions calls and workers defined in the function body has changed. For more information, refer to the [concurrency section].

- The `fork/join` statement has been removed. You can use the new `fork` statement to start multiple workers in parallel with each other. Each worker name becomes a variable of type `future<T>` where T is the return type of the worker. You can use the new `wait` action to wait for one or more workers. For more information on the fork statement, wait action, and other concurrency-related changes, refer to the [concurrency section].

- The `done` statement has been removed. Now workers can return values using the `return` statement.  For more information, refer to the [concurrency section].

- The `await` statement has been replaced by the new `wait` statement. For more information, refer to the [concurrency section]. 

- The `foreach` statement has been changed in a consistent way to match with type binding patterns. Here is the old syntax. 

```ballerina
map<Student> class = { ... } 
foreach name, std in class {
    string msg = name + " details: ";
    io:println(msg, std);
}

```

Now the `foreach` statement allows the value to be destructured, while iterating over a sequence. Here is an example. 

```ballerina
foreach (string, Student)  (name, std) in class {
    string msg = name + " details: ";
    io:println(msg, std);
}

```

- Additionally, `list` type iteration provides only the value as an argument. It no longer gives the index variable as the first argument. JSON is no longer iterable, instead, use array of JSON (`json[]`) or map of JSON (`map<json>`). For more information, refer to the [binding pattern section].  

- Iterable Operations (`foreach`, `map`, `filter`) signatures have been changed to be consistent with the `foreach` statement arguments.

# What's new in Ballerina 0.990.0

This release contains significant changes and new features in areas such as structural types, error handling, concurrency, objects, and endpoints.

## Structural Types

Ballerina has a structural type system in which compatibility of types and values is determined based on their structure; this is particularly useful in integration scenarios that combine multiple, independently-designed systems. The following changes to the Ballerina type system encourage data-oriented thinking, provides the foundation for safe concurrency by introducing immutable values, makes structural types, and mutability work together smoothly.  

### Pure values

A value is considered as a pure value if it is a simple value (`nil`, `boolean`, `int`, `float`, `string`) or a structured value that contains only pure members. Error values are also considered as structured values. 

### anydata type

The `anydata` type consists of pure values whose `basic` type is not `error`. Thus `(anydata|error)` contains all pure values. So `anydata` is equivalent to `() | int | float | decimal | string | (anydata|error)[] | map<anydata|error> | xml | table`.

Ballerina records are by default open. In the previous release, `record {}` means `record { any … }`. With the introduction of pure values now `record {}` means it is open only to pure values. 

### clone() built-in method

Performs a deep copy of a pure value. It is a no-op operation on simple values and error values. This operation is guaranteed to succeed on pure values and preserves cycles, inherent type, and immutability of values. 

### Record type referencing

Record type referencing is a mechanism for copying the fields from one record type descriptor into another record type descriptor. 

### Value equality and reference equality 

From this release onwards, we have changed the semantics of operator `==` to perform a deep value equality only on `anydata` values. A compile time error occurs if used for some other purpose. This operator is not allowed on error values.

We have introduced a new operator `===` that performs reference equality on reference types and is otherwise same as `==`. The `!==` operator results in the negation of the result of the `===` operator. 

### Immutable values

We have introduced immutable values from this release onwards. Immutability is not part of the static type system in the language. It exists only in the runtime. You can use the `freeze` built-in method to make a structured value frozen at runtime. This method sets the `frozen` flags in the value. A `frozen` structured value cannot be modified afterward and any such attempts will results in panics. 

The `freeze` built-in method changes the value to which it is applied to be frozen and returns that value. It works in a similar way to the `clone` method.

There is a `isFrozen()` built-in method on all structural basic types that returns `true` or `false` depending on whether the value is frozen. 

```ballerina
person p1 = {name:"John", age:34};
person p1Frozen = p1.freeze();
boolean b = p1 === p1Frozen; // true

```

Structural typing and mutability
is and is-like 
Type test expression
Type assertions
Type guards
convert()
stamp()
match statement 

The existing match has been changed in this release from a type match to a value match statement. The match statement is a value switching construct that allows selective code execution based on the value of the expression that is being tested.

```ballerina
match variable {
	“Bal” => io:println(“Matched a String with value Bal”);
	0 => io:println(“Matched an int with value 0”);
	10 => io:println(“Matched an int with value 10”);
	(15, “Lang”) => io:println(“Matched a tuple with (15, “Lang” values”);
	false => io:println(“Matched a boolean with value false”);
	{a: 15, b: “Lang”}=> io:println(“Matched a mapping value”);
	_ => io:println(“Matched to Default Pattern”);
}

```

The `match` statement can also be used to match the structure of a value by using binding patterns. The matched value will be destructured to new variables within the scope of the match pattern. Here is an example of the structured value match.

```ballerina
match variable {
	var (var1, var2) => { // match a tuple value with two members
io:println(var1); 
io:println(var2); 
}
	var {a: var1, b: var2} => { //match a mapping value with fields “a” and “b”
io:println(var1); 
io:println(var2); 
}
	var x => io:println(“Matched to Default Pattern”); //all values will match
}

```

## Error Handling

Ballerina distinguishes two kinds of errors; normal and abnormal errors. Abnormal errors are unusual occurances and they are usually signs of bugs or programmer errors. It is usually not possible to recover from abnormal errors. Normal errors, on the other hand, are recoverable and they are part of your business logic. 

Normal errors can be returned from functions, workers and resources hence have an explicit control flow. They are also statically type checked. Abnormal errors stop the execution of a program unless they are trapped: implicit control flow. Following changes and improvements defines consistent policy on error handling throughout the language. Also, these changes have an impact on concurrency features which is explained in the concurrency section.

### The error type

The error type is a structured basic type which is used only for representing errors. A value of error type contains a reason which is a string identifier for the category of error, a detail which is a frozen mapping providing additional information and a stack trace. Error values are made immutable from this release onwards, since they can propagate across concurrent execution blocks. Allowing them to be mutable can be dangerous. 

Here is an example for usage of error type. The function `getAcctBalance` returns a union of `decimal` and `error<string, AccountData>` types.

```ballerina
type AccountData record {
    string accountID;
};

function getAcctBalance(int id) 
returns decimal | error<string, AccountData> {...}

```

The `any` type was the union of all types in Ballerina. However, from this release onwards, we have removed the `error` type from `any`. This change forces errors to be documented explicitly, even if a function returns `any`. 

### Check expression

The `check` expression is a useful unary operator in the language, which allows you to get an error out of the way in arbitrary expressions. If the type of the expression E is `T | error`, then the type of expression check E will be `T` if it evaluates to `T`. If the expression evaluates to error, then the function terminates immediately as if by `return E`. The `check` operator forces you to declare error as part of the return type of your function. 

In the previous release, check throws the error if the function does not have error type as part of the `return` type. This causes normal errors to become exceptions or abnormal errors due to negligence of the programmer. 

Here is a sample usage of check operator.

```ballerina
function startHTTPListener() returns Listener | error {
    string strVal = getValue(HTTP_PORT);
    // int.convert() has the type 'int | error'
    int port = check int.convert(strVal);
    ...
}

```

### Panic statement

This is a replacement for the `throw` statement to allow programmers to create a panic, which causes the execution to stop. Function calls will unwind until the panic is stopped by trapping it and converting it into an error. Stack trace is recorded in the error value at the time of panic. 

```ballerina
int | error i = int.convert("4t4");
if i is error {
    panic i;
}

```

### Trap expression

The `trap` expression stops a panic and gives access to the error value associated with the panic. This is a replacement for the `try-catch` statement. In `var a = trap E`, the type of the variable `a` will be `T | error` where `T` is the type of the expression `E`.

## Binding Patterns

Binding patterns are used to support destructuring, which allows different parts of a single structured value each to be assigned to separate variables at the same time. Binding patterns can be used both with or without a type. Using a `typed binding pattern` will assign the destructured values to new variables, while a `destructure assignment binding pattern` will assign the destructured values to existing variable references.

This release introduces two binding patterns; tuple binding pattern and record binding pattern.

Tuple binding pattern destructures a `tuple` value based on the number of members in the `tuple` value. Here is an example of a tuple binding pattern.

```ballerina
(int, string) tupleVar = (20, “Ballerina”);
(int, string) (intVariable, stringVariable) = tupleVar; //typed binding pattern
io:println(intVariable + “ “ + stringVariable);
int intVariable2;
string stringVariable2;
(intVariable2, stringVariable2) = tupleVar; //binding pattern with variable references
io:println(intVariable2 + “ “ + stringVariable2);

```

Record binding pattern destructures a mapping value based on the presence of a field name. Here is an example of a record binding pattern. In this example a `Person` typed record with fields `name` and `age` will be destructured. The variable name to which the value is to be assigned can be given against a field name following a colon. Omitting a variable name will result in using the same name as the field name.

```ballerina
Person {name, age: personAge} = getPerson(); //typed binding pattern
io:println(name + “ “ + personAge);
string personName;
int age;
{name: personName, age} = getPerson(); //binding pattern with variable references
io:println(personName + “ “ + age);

```

## Concurrency

Ballerina has built in support for concurrency. `worker` and `start` creates new concurrent flows. `wait`, `->` (send), `<-` (receive), and 'flush' constructs offer flow control and communication between concurrent flows. `lock` construct can be used to achieve safe concurrent access.

Ballerina execution model is based on 'strands' that are lightweight execution flows. Strands may run parallely and scheduled by the runtime. Strands are represented by `futures` in the language. 

### Start

Start executes a given function in a new strand, resulting a new `future`. This `future` can be used to interact with the concurrent execution flow.

### Workers and Fork

Workers offer a way to write a concurrent code section inline within an existing function.

A function has one or more workers including an unnamed default worker and other named workers. These workers are peers of each other. The termination of the function is independent of the termination of its named workers. Returning from a named worker terminates that worker, but does not cause a return from the function. Workers have same return declaration as functions which defaults to ()

The workers created by a `fork` statement are peers of each other: they can exchange messages with each other but not with any other workers. In other respects, workers created by a `fork` statement behave the same as workers created by a function body.

### Futures and wait

A `future` is just a worker that can be waited for outside the function that starts the worker. 

A wait-action waits for one or more futures to be terminated and makes available their return value (including any raised errors). Panic in waited-for future should result in panic in `wait`. Waiting for a future and waiting for a worker are conceptually the same. 

Using the future you can (a) cancel the strand (b) wait for the strand return.

### Send Receive and Sync Send

Send/sync-send will send `anydata` type information to another worker, in which `receive` can be used in the other worker to retrieve the message. Compile time analysis guarantees there is a matching receive for each send.

If the data is not available, receive will wait, similarly if the receiver is not reached yet, the sync send will wait. Both sync send and receive will propagate whichever error that happened in the opposing strand to the current strand.

`flush` will wait until all the messages are sent from the current worker to the specified worker. If the receiver failed or panicked, then that error will be propagated to the waiting strand.

### Lock

Lock can be used to make the concurrent access safe for a set of variables. It creates two phase deadlock-free synchronization guaranteeing unique access to each variable.

```ballerina
public function invokeWorkers() returns int {
    worker w1 {
        int i = 5;
        i -> w2;
    }
    worker w2 returns int {
        int j = <- w1;
        return j;
    }
    return (wait w2) + 1;
}
```

## Variable Initialization

Compile time constants
Final variables
Regular variables
structure members

## ‘decimal’ type

Decimal type is a basic, simple type in Ballerina. The decimal type corresponds to the 128-bit IEEE 754-2008 decimal (radix 10) floating point number format. A decimal value has 34 decimal digits of significand and an exponent range of −6143 to +6144.

Syntax:
decimal d = 4.565;
Defining a decimal variable is very similar to defining a float variable in Ballerina
By default, all the floating point literals will be treated as float values unless otherwise we explicitly state as decimal

Requirements:
To represent high precision and high scale numbers
Current float type (64 bits) may not be adequate in certain cases 
Financial and commercial use cases
Currency representations
To resolve floating point precision error problem exists in float type
Floating-point numbers cannot precisely represent all real numbers
Hence, it leads to many erroneous results when performing arithmetic operations
For example, the below boolean expression would result false when the numbers are represented using float type
       0.1 + 0.2 == 0.3 
// Above would result `false` if the numbers are represented using `float` 
This is because of the non-representability of 0.1 (0.0001100110011..) and 0.2 (0.001100110011..) in binary
 IEEE 754-2008 Decimal128 format resolves this problem by treating the radix 10 when representing the values 

Basic functionalities:
Addition
Multiplication
Division
Subtraction
Modulus
Negation
Type conversions (float → decimal, decimal → float, int → decimal, decimal → int...)
Comparisons (==, !=, >, <, >=, <=)

#### length
#### isNaN
#### isFinite
#### isInfinite

# Experimental Language Features 

--experimental
This section should contain all the language features that we tag as experimental for 1.0.

## Micro-Transactions

###Transaction initiator statement

`oncommit` and `onabort` function handlers are removed from the transaction statement and are substituted with committed and aborted blocks.

```ballerina
transaction  with retries=2{
	_ = testDB->update("Insert into Customers (firstName,lastName,registrationID,creditLimit,country)
                        	values ('James', 'Clerk', 200, 5000.75, 'USA')");
	_ = testDB->update("Insert into Customers (firstName,lastName,registrationID,creditLimit,country)
                        	values ('James', 'Clerk', 200, 5000.75, 'USA')");
} onretry {
	returnVal = -1;
} committed {
	committedBlockExecuted = true;
} aborted {
	abortedBlockExecuted = true;
}
```

### Transaction participants

Functions and resource functions can be marked as transaction participants using @transactions:Participant {} annotation from the transactions module. This annotation facilitate registering handler functions that are triggered when the transaction initiator commit or abort.

And panic propagating out of such participant is considered as transaction participant failure when deciding to commit the transaction.

With this introduction of transaction participants, nesting transaction blocks are prohibited, both statically and at runtime.

#### Local participants

```ballerina
@transactions:Participant {
	oncommit:commitFunc,
	onabort:abortFunc
}
public function participantFoo() {
string trxId = transactions:getCurrentTransactionId();
	io:println("In transaction participant of: " + trxId);
	// Simulate participant failure.
	error e = error("An error");
	panic e;
} 
```

Athe above code will cause the calling transaction to fail due to the panicking of a participant.

If the participantFoo is called without a transaction, all the transaction related annotations will be ignored for that particular call.

`oncommit` and `onabort` functions should have the signature of `function (string trxId)`

#### Remote participants

Similar to local transactions we can also annotate resource functions so that they are considered transaction participants. Annotated transaction participants are not allowed to have transaction blocks similar to local participants.

Panic in remote participants will result in transaction failure.

```ballerina
 @transactions:Participant {
oncommit:commitFunc,
	onabort:abortFunc
}
resource function testSaveToDatabaseSuccessfulInParticipant(http:Caller ep, http:Request req) {
	saveToDatabase(ep, req, false);
}

```

# Standard Library

## HTTP Listener/Client

With endpoints and service syntax changes, HTTP endpoint and service are changed as follows:

HTTP Listener:

```ballerina
listener http:Listener httpEp = new(9090, config = {
    secureSocket: {}
});
```

> **Note**: Listener endpoint port is the first argument which is a required parameter and other listener configurations go under the second argument. 

Service and resource definition:

```ballerina
service hello on httpEp {
	resource function sayHello(http:Caller caller, http:Request req) {
    	_ = caller->respond("hello ballerina!");
    }
}
```

Service can get attached to a listener endpoint that is declared inline as follows.

```ballerina
service hello on new http:Listener(9090) {
	//...
}

```

Client endpoint:

```ballerina
http:Client clientEndpoint = new("https://localhost:9092", config = {
    secureSocket: {}
});

```

> **Note**: URL is the first argument which is compulsory and other endpoint configurations go under the second argument.

## Load Balancer

The Load Balancer exposes an abstract object to allow users to customize the routing decision making(load balancing rule) for special purposes. Only one rule is active at any time and it is specified with the lbRule in the `LoadBalanceClientEndpointConfiguration` record. By default, round robin rule is used as the load balancing strategy.

Load balancer client endpoint syntax goes as follows.

```ballerina
http:LoadBalanceClient customLoadBalanceClient = new ({
   targets: [
       { url: "http://localhost:8081/mock" },
       { url: "http://localhost:8082/mock" },
       { url: "http://localhost:8083/mock" },
   ],
   lbRule: customLbRule
});

```

## TCP client/server

TCP server and client are completely redesigned with this release. Now users can use the TCP server similar to the WebSocket listener where there are set of predefined resources to invoke based on actions that take place. TCP client can be used to connect to a remote TCP server for data sending purpose. Additionally, callback service can be attached to the client if there is a data retrieval use case.

```ballerina
import ballerina/socket;

```

Listener:

```ballerina
listener socket:Listener server = new ({ port:<PORT_NUMBER> });
service echoServer on server {
    resource function onAccept (socket:Caller caller) { }
    resource function onReadReady (socket:Caller caller, byte[] content) { }
    resource function onClose(socket:Caller caller) { }
    resource function onError(socket:Caller caller, error er) { }
}

```

Client:

```ballerina
socket:Client socketClient = new({host: "localhost", port: 54387, callbackService: ClientService});
byte[] payloadByte = “Hello Ballerina”.toByteArray("UTF-8");
_ = socketClient->write(payloadByte);

service ClientService = service {
    resource function onConnect(socket:Caller caller) { }
    resource function onReadReady (socket:Caller caller, byte[] content) { }
    resource function onClose(socket:Caller caller) { }
    resource function onError(socket:Caller caller, error er) { }
};

```

`socket:Client/socket:Caller` has `shutdownRead` and `shutdownWrite` functions available in addition to the write.

## MySQL Client
MySQL client syntax is changed as follows.

```ballerina
mysql:Client testDB = new({
        host: "localhost",
        port: 3306,
        name: "testdb",
        username: "root",
        password: "root",
        poolOptions: { maximumPoolSize: 5 },
        dbOptions: { "useSSL": false }
    });
```

## H2 Database Client
Compile time validation for InMemory, Server and Embedded Mode configurations are introduced along with the
changes in client syntax changes.

Creating a client in H2 Embedded Mode
```ballerina
h2:Client testDB = new({
        path: "/home/ballerina/test/",
        name: "testdb",
        username: "SA",
        password: "",
        poolOptions: { maximumPoolSize: 5 }
    });
```

Creating a client in H2 Server Mode

```ballerina
h2:Client testDB = new({
        host: "localhost",
        port: 9092,
        name: "testdb",
        username: "SA",
        password: "",
        poolOptions: { maximumPoolSize: 5 }
    });
```

Creating a client in H2 In-Memory Mode

```ballerina
h2:Client testDB = new({
        name: "testdb",
        username: "SA",
        password: "",
        poolOptions: { maximumPoolSize: 5 }
    });
```

## gRPC
gRPC is a protocol that is layered over HTTP/2 and enables client and server communication by combination of any supported languages. In gRPC, a client application can directly call methods on a server application on a different machine, making it easier for you to create distributed applications and services.

With endpoints and service syntax changes, gRPC endpoint and service are changed as follows:

Listener (inbound) endpoint:

```ballerina
listener grpc:Listener grpcEp = new (9095, config = {
	secureSocket: {}
});

```

> **Note**: Listener endpoint port is the first argument and other listener configurations go under the second argument. 

Service and resource definition:

```ballerina
service Hello on grpcEp {
    resource function hi (grpc:Caller caller, string req) {
        _ = caller->send("Hello, World!");
    }
}

```

Client endpoint:

```ballerina
HelloClient helloEp = new("https://localhost:9090", config = {
    secureSocket: {}
});

```

> **Note**: Server URL is the first argument and other endpoint configurations go under the second argument.

The contract first approach was introduced for gRPC service creation, which means you first define the service contract (.proto file). From the service contract, you generate stub and sample Ballerina service using the built-in proto compiler. Earlier we followed a code first approach to create a gRPC service, but we depreciated it due to a limitation in ensuring backward compatibility. Instead we recommend contract first approach to create a gRPC service.

## WebSocket

With endpoints and service syntax changes, WebSocket endpoint and service are changed as follows:

WebSocket Listener:

```ballerina
listener http:WebSocketListener helloListener = new(9095, config = {secureSocket: {}});
service onTextString on helloListener {
	resource function onText(http:WebSocketCaller caller, string data, boolean finalFrame) {
    	var returnVal = caller->pushText(data);
           }
}

```

WebSocket Client:

```ballerina
http:WebSocketClient wsClientEp = new(REMOTE_BACKEND_URL, config = { callbackService: clientService });
service clientService = @http:WebSocketServiceConfig {} service {
	resource function onText(http:WebSocketClient caller, string text) {
    		var returnVal = caller->pushText(data);	
	}
}

```

## WebSub

WebSub endpoint and service are changed as follows with the new endpoints and service syntax change

Listener:

```ballerina
listener websub:Listener websubEP = new(8181);
```

Service and resource definition:

```ballerina
service websubSubscriber on websubEP {
    resource function onNotification(websub:Notification notification) {
        log:printInfo("WebSub Notification Received");
    }
}
```

WebSub Hub start up function signature is improved to accept parameters under two categories. 
It accepts an `http:Listener`, on which the hub service will start on, as the first parameter, 
and a `HubListenerConfiguration` record as a defaultable parameter to specify hub related configuration.  

```ballerina
var webSubHub = websub:startHub(new http:Listener(9090), hubConfiguration = {
            leaseSeconds: 86400
    });

```

Improved the hub client by extending configuration to have the complete functionality of the HTTP Client.

```ballerina
http:ClientEndpointConfig conf = {
    followRedirects: { enabled: true, maxCount: 5 }
}; 

websub:Client websubHubClientEP = new("https://localhost:9191/websub/hub", config = conf);

```

## Messaging

The `ballerina/jms` module provides an API to connect to an external JMS provider like Ballerina Message Broker or ActiveMQ. This module provides different consumer and producer endpoint types for queues and topics. The endpoints prefixed with “Simple” will automatically create a JMS Connection and a JMS Session when the endpoint is initialized. For other endpoints, the developer must explicitly provide a properly initialized JMS Session.

New syntax for JMS message producer and message receiver is as shown below. 

> **Note**: JMS Connection and session definitions remain unchanged. 

```ballerina
jms:Connection conn = new({
        	initialContextFactory:<provider specific initial context factory>,
        	providerUrl: <provider url>
});

jms:Session jmsSession = new(conn, {
        	acknowledgementMode: "AUTO_ACKNOWLEDGE"
    	});

```

JMS message producer definition:

```ballerina
jms:QueueSender queueSender = new({
        session: jmsSession,
        queueName: "MyQueue"
});

```

JMS message receiver, service, and resource definition: 

```ballerina
listener jms:QueueReceiver consumerEndpoint = new({
       	 session: jmsSession,
        	 queueName: "MyQueue"
});

service jmsListener on consumerEndpoint {
resource function onMessage(jms:QueueReceiverCaller
                                      consumer, jms:Message message) {
            }
      }
      
```

# Tooling

## IDEs and Language Intelligence

Added test generation code action for services and functions.

## Design View

API Designer view was added to edit/design HTTP APIs. Use “Show API Designer” command to activate API Design view in VSCode.

## Network logs inspector

## Compatibility

Language Core introduces experimental features and in order to support code intelligence such as auto completion in VSCode Plugin. Enable the `Allow Experimental` setting in the user settings to experience this.

# Other Improvements

## Extensions

### Kubernetes

The `ballerinax/kubernetes` module now supports generating Istio gateways and Istio virtual services for Ballerina services. These artifacts can be generated with `@kubernetes:IstioGateway` and `@kubernetes:IstioVirtualService` annotations, which allow to deploy services seamlessly to Istio.

All artifacts are now generated into a single YAML file by default. This can be overridden by setting `singleYAML: false` in the `@kubernetes:Deployment` annotation.

# Performance results

Please refer Ballerina [performance test results](https://github.com/ballerina-platform/ballerina-lang/blob/v0.990.0/performance/benchmarks/summary.md) available in the repository.

# Bug Fixes

Please refer to [GitHub milestone issues](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.990.0+is%3Aclosed+label%3AType%2FBug) to view bug fixes

# SPECIFICATION DEVIATIONS IN 0.990.0 RELEASE

## Values,Type and Variable

### Nil

Use of “null” value in a non-JSON related context is not restricted yet.
An array that contains a nil value has an invalid string representation. 

### Int

Binary Literal are allowed as an integer literal. But it is not part of the 0.990.0 language  specification. This support to be removed from the Compiler in future.

### Float and Decimal

Two NaN values with different bit patterns are not considered as same at the runtime, but the specification mentioned otherwise. 
In Decimal implicit initial value taken as “0” instead of “+0.0”
For floating point, === means the same member of value space. For example,
NaN === NaN, -0.0 !== +0.0 But this validation gives incorrect results at runtime. 

### String

String values are not iterable in this release.

### Record

A Required field of a record can have a default value. But 0.990.0 language specification does not allow default values. If a record mapping constructor doesn’t have a value given for such field, then the default value is taken. 

### Table

Table values are not iterable using foreach statement.

### Error

Detail value of an error is not frozen.
The runtime implementation doesn’t enforce the "detail" value of an error to be a subtype of map<anydata|error>.

### XML

In an XML sequence, a character set is treated as a single string. However, in the specification, each character item is represented by a string with a single code point.

### Object

Error handling in the object initialization method is not supported yet. Only nil returning `__init` methods are allowed. 

This prevents using "check" expression inside the object initialization function.

Similar to record fields, an Object field can have a default value. Built-in Iterator, Iterable, and Collection abstract object type are not supported yet.

### Singleton Types

Runtime allows float values as a singleton type.

Defining a variable with singleton type is not allowed. E.g., `5 a = 5`

### Anydata

In the implementation, a structured value that contains an error value is not considered as "anydata".

### Const

An integer constant reference can’t be used to define the length of an array.

### Freeze

The compiler doesn't do data flow analysis for frozen value. This results in a panic at run-time if a value is updated after a freeze. 

`'freeze()` and `isFrozen()` are not allowed on nil value.

### Functions

Falling off an end of a function body will not implicitly return () if the function returns an optional type. Explicit return () statement is required.

## Expressions

In this release, Const support only added for Boolean literals, int literals, floating point literals, and string literals. 
The current implementation uses different interpolation syntax “{{expr}}” instead of “${expr}” in string templates
Decimal division and remainder operations panic if the second operand is 0.0;.
Iiterator, unfrozenClone, stackTrace built-in methods are not supported yet.

## Binding Patterns

Error binding pattern is not supported in this release.

# Getting Started
You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour](https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).
