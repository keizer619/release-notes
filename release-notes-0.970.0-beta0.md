# Overview to Ballerina 0.970.0-beta0
This release packages significant enhancements to Ballerina language syntax and type system.

The enhancements include the introduction of object type where you can either define the methods of the object within the definition itself or you can bind them from outside the object.
```ballerina
public type Response object {
   public {
       @readonly int statusCode;
       string reasonPhrase;
       string server;
       ResponseCacheControl cacheControl;
   }

   private {
       int receivedTime;
       int requestTime;
   }

   public function setStatusCode(int statusCode);

}

//when binding to the object
public function Response::setStatusCode(int statusCode) {
    self.statusCode = statusCode;
}
```

We have introduced  union type in this release. Union types are types whose set of values is the union of the value spaces of its participant types. Note that with the introduction of union and type switching, we no longer need type casting
```ballerina
type json (string | float | int | boolean | null | map<json> | json[]);
```

The match statement is introduced to help manage union constructs. Unions have to be matched and tuples have to be destructured using the match statements.
```ballerina
function getPersonFromDB(string name) returns (string,int,float)|error {
	// Do a DB lookup and load the person from the table
	// If an error occurs while doing the DB lookup, return error
}

var r = getPersonFromDB(“sam”);
match r {
	(string name, int age, float salary) => { 	
		io:println(name);
		io:println(age);
		io:println(salary);
	}
	error err => {
		io:println(“Error occurred while loading person:” + err.message);
	}
}
```

A `but` expression is also introduced facilitate expression evaluation. If the value of the evaluated expression matches to some given type, then we can update the result for a prefered value.
```ballerina
int x = p but {
string => 1,
float f => <int> f
}
```

The safe navigation operator has been enhanced to allow safe navigation in the event of null or an error. This will be useful with maps that now have error as common response status.
Nil-lifting navigation has been introduced to help eliminate null pointer exceptions.
Error-lifting navigation has been introduced where you can lift errors when navigating through fields, using the “!” operator.
``ballerina
type  Person {
    Info|() info;
};

type Info {
    Address|() address;
};

type Address {
    string street;
    string city;
    string country = "Paradise";
};

// nil lifting
// in the following code even if p is nil,
// it will not throw a runtime NPE, but instead y’s value will be nil.
function foo () {
    Person|() p;
    string|() y = p.info.address.city;
}

// error lifting
// in the following code, if the value of p.info or p.info!address is 'error',
// then the value for the whole rhs expression will be error.
function foo () {
    Person p = something;
    string|() y = p.info!address!city;
}
```

This release has replaced const with @final for defining constants.
```ballerina
@final string TXN_STATE_ACTIVE = "active";
```

This release has also replaced enums with types, where you can define a new type with a pipe (‘|’) separated list of values.
```ballerina
type TransactionState "active" | "prepared" | "committed" | "aborted";

@final TransactionState TXN_STATE_ACTIVE = "active";
@final TransactionState TXN_STATE_COMMITTED = "committed";
```

An error elimination operator has been introduced. You can use the operator if there are errors returned from a function, they will be thrown. You need to use this with caution when you are absolutely sure that an error won’t be returned or the error will be handled by upstream code.
```ballerina
int x =? foo();
```

A new concept of nil has been introduced. Note that nil is not the same as null. null should be avoided in general and we have to use nil, which is denoted with , (). null is now only used with JSON type, because, possible set of values for JSON is a union of several types including null.
```ballerina
Person? p = getPerson(“john”); // NOTE: Person? Is a syntactic shortcut for Person|()
match p {
	Person p1 => {
		string name = p1.name;
		io:println(name);
     }
     () => {
		io:println(“Person does not exist”);
     }
}
```

This release also introduced first class support for streaming event processing. It allows you to build streaming queries with user friendly syntax. The queries include projection, filtering, windows, stream joins ,and patterns.
```ballerina
//Create a stream that is constrained by the StatusCount struct type.
stream<StatusCount> filteredStatusCountStream;

//Create a stream that is constrained by the Teacher struct type.
stream<Teacher> teacherStream;

// Create a forever statement block with the respective streaming query.
// Write a query to filter out the teachers who are older than 18 years, wait until three teacher
// object are collected by the stream, group the 3 teachers based on their marital status,
// and calculate the unique marital status count of the teachers.
// Once the query is executed, publish the result to the `filteredStatusCountStream` stream.
forever{
        from teacherStream where age > 18 window lengthBatch(3)
        select status, count(status) as totalCount
        group by status
        having totalCount > 1
        => (StatusCount [] status) {
                filteredStatusCountStream.publish(status);
        }
    }

```


# Improvements
## Language & Runtime
- `const` keyword has been replaced with `@final`
- `enum` concept is replaced with ‘type` creation
- Introduction of `object` type, union types and nil type
- Introduction of `@readonly` annotation
- Safe navigation support, introduction of  error elimination operator, match` and `but` statements
- Multiple value return type function concept is removed and replaced with the ability to return tuples
- Inline tuple destructuring capability
- Introduction of streaming capabilities, table literal support, transaction commit/failure callbacks
- Introduction of fail statement
- Removal of transformer syntax

## Standard Library

### Config API
  - Improved API with support for retrieving configs of different data types
  - Support for securing sensitive data

### HTTP
  - Support for HTTP caching, HTTP access logs, connection throttling and HTTP version 2 (HTTP/2)
  - Support for publishing HTTP trace logs to sockets and files
  - Support for service versioning and per service chunking configuration
  - Improved APIs for HTTP header related operations and OCSP stapling for checking the revocation status of certificates
  - Introduce a new method called `setPayload()` to http request and response to take any (string, xml, json, blob, io:ByteChannel or mime:Entity[]) type of payload

### MIME
  - Introduce a new method to get the base type from a given content type and a new method to set any(string, xml, json, blob,io:ByteChannel or mime:Entity[]) type of body to entity
  - Improve entity body operations to rely on content type and APIs for MIME specific base64 encoding / decoding

### WebSocket
  - The WebSocket upgrade resource has been moved to the Http Service
  - A new function called acceptWebSocketUpgrade has been introduced to the http:Listener
  - The WebSocket resource signatures have endpoints and not WebSocketConnector as the first argument.
  - The WebSocket resource signatures now have basic data types instead of the frames in previous implementations.

### WebSub
  - Support for [WebSub][3] subscriber, hub and publisher to facilitate communication between publishers and subscribers of Web content based on HTTP web hooks

### Observe
  - Support for observability through metrics and tracing APIs

### SQL
  - Support MySQL and H2 packages to interactions with respective DBs.
  - Add mirror table support to read write directly via tables

### Util
  - Improved APIs for base64 encoding and decoding

## IDEs & Language Server

### Language Server
  - Support find all references, go to definition and hover provider support for match expression, tuple support, union type support, final and readonly variables, endpoint, new service syntax, and object and record statements.
  - Support documentation syntax, annotation syntax and completion support for records, objects and endpoints syntaxes

### Composer
  - Introduction of trace log analyzer tool
  - Support match statement visualization in diagram and source generation support for match statement, transaction statement
  - Removal of transformer UI

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
[3]: https://www.w3.org/TR/websub/
