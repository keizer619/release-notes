# Overview to Ballerina 0.975.0

Ballerina 0.975.0 is an iteration for Ballerina 0.970.0, which was released previously. This release has significant improvements to the standard library and IDE support. Apart from that, there are also some improvements made to the language syntax. Introduction of new BALO format has improvements on compilation time as well.

# Compatibility and Support

## New int range expression

Previously `[x .. y]` syntax was used to represents an integer range in a `foreach` statement. This syntax is now `x ... y` in Ballerina 0.975.0.

Old syntax (0.970.1)
```ballerina
foreach val in [1 .. 5] {
   io:println(val);
}
```

New syntax
```ballerina
foreach val in 1 ... 5 {
   io:println(val);
}
```

## Syntax change in record type definitions

The record type definition syntax now requires a `record` keyword. The syntax used previously also works on Ballerina 0.975.0, but this will not be supported in future releases.

Old syntax (0.970.1)

```ballerina
type Person {
   string name,
   int age,
};
```

New syntax

```ballerina
type Person record {
   string name,
   int age,
};
```

## Syntax change for the next statement

The `next` statement is renamed to `continue`.

Old syntax (0.970.1)

```ballerina
while (i < 5) {
   i = i + 1;
   if (i == 2) {
       next;
   }
}
```

New syntax

```ballerina
while (i < 5) {
   i = i + 1;
   if (i == 2) {
       continue;
   }
}
```

## New blob literal support

`base16` and `base64` based literal values are the two types of literal values supported for blob. The `base16` type has a base 16 based character sequence and the `base64` type has a base 64 based character sequence or padded character sequence.  

New syntax

```ballerina
blob a = base16 `aaabcfccad afcd34 1a4bdf abcd8912df`;  
blob b = base64 `aaabcfccad afcd3 4bdf abcd ferf =`;
```

## Variable shadowing removed

Now variables cannot be shadowed and are required to have unique names. The only exception is in XML namespaces that already have variable shadowing support.

Old syntax (0.970.1)

```ballerina
string name;

type User object {
    private {
        string name;
    }

    function setName(string name) {
        self.name = name;
    }
};

function print(string name) {

}
```

New syntax

```ballerina
string name;

type User object {

    private {
        string userName;
    }

    function setName(string n) {
        self.userName = n;
    }

};

function print(string n) {

}
```

# Improvements

## Language & Runtime

- Blob literal support
- Make parentheses optional for 'if' and 'while' statements
- Integer range binary operators to create arrays of integers
- Syntax change in record type definitions.
- Variable shadowing support removed.

## Standard Library

- Add capability to read/write fixed signed integer, float, boolean and string values via data IO APIs
- Error handling support for WebSocket service

```ballerina
@http:WebSocketServiceConfig {
   path: "/error/ws"
}
service<http:WebSocketService> errorService bind { port: 9090 } {
 onError(endpoint ep, error err) {
       io:println(string `error occurred: {{err.message}}`);
   }
}
```

- Allow different types of payloads to be used directly with HTTP client actions and response calls. E.g :-

```ballerina
response = clientEP->post("/test", xml `<color>Red</color>`);
response = clientEP->post("/test", { name: "apple", color: "red" });
_ = caller -> respond("Hello World!");
```

- Improve redirect functionality so that it supports both HTTP and HTTP2
- Support event notification payloads of different content types with WebSub
- HTTP Name based virtual hosting support.

```ballerina
        @http:ServiceConfig {
   basePath:"/page",
   host:"abc.com"
}
```

- Improve HTTP error handler to recover from source connection failure.
- Enhance the API to control the circuit breaker status as per user requirements.

```ballerina
 http:CircuitBreakerClient cbClient = check <http:CircuitBreakerClient>clientEP.getCallerActions();
        if (counter == 2) {
            cbClient.forceOpen();
        }
```

## Build & Package Management

- Support to build reproducible builds using the lock file.

## IDEs & Language Server

### Composer

- HTTP Trace Log Improvements in Composer.

### Language Server

- Add initial Indexing Support for Language Server.
- Improve Language Server Error Reporting.
- Add Renaming Support for Variable Symbols.
- Signature Help Support for Action Invocations.
- Match Expression Completion Support with Snippets.
- Completion Support for Function Invocation Scope.
- Code Action Improvements with Generate Function for Undefined Functions.

### IntelliJ IDEA

- User repository support.
- Package auto-import support.
- New live templates.
- New code inspections.
- Performance improvements.

### VSCode

- Syntax highlighting improvements.

# Bug Fixes

From previous stable 0.970.1 release, bug fixing has been conducted in few release iterations. Please refer following github milestones of bug fixes.

- [0.971.0](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+milestone%3A0.971.0)
- [0.972.0](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+milestone%3A0.972.0)
- [0.973.0](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+milestone%3A0.973.0)
- [0.974.0](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+milestone%3A0.974.0)
- [0.975.0](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aclosed+label%3AType%2FBug+milestone%3A0.975.0)

# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour](https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).
