# Overview to Ballerina 0.982.0

This release includes significant language syntax improvements and also includes improvements introduced via bug fixes to stabilize the language.

In addition to language improvements, this release also introduces the initial version of the Ballerina Native feature that allows simple Ballerina programs with the main function to be compiled into native executables.

# Compatibility and Support

Ballerina Message Broker has been removed from the ballerina-platform distribution. Therefore, broker execution scripts are no longer available under the bin directory. However, you can still download the Ballerina Message Broker distribution from [Github repository releases](https://github.com/ballerina-platform/ballerina-message-broker/releases) if necessary.

See the following section for details on syntax changes:

## Syntax changes

### New documentation syntax

Old syntax:

```ballerina
documentation {
    Adds parameter `x` and parameter `y`.
    P{{x}} one thing to be added
    P{{y}} another thing to be added
    R{{}} the sum of them
}
function add(int x, int y) returns int { return x + y; }
```

New syntax:

```ballerina
# Adds parameter `x` and parameter `y`.
# + x - one thing to be added
# + y - another thing to be added
# + return - the sum of them
function add(int x, int y) returns int { return x + y; }
```

Note: Ballerina does not support the old documentation syntax with the new release. 

### Reorder documentation in resources

In previous versions, documentation was added after annotation attachments. With the latest changes, documentation should be added before the annotation attachment.

Old syntax:

```ballerina
service<http:Service> update_token bind { port: 9295 } {

    @http:ResourceConfig {
        methods:["GET"]
    }
    # Updates the access token.
    #
    # + caller - Endpoint
    one_px_image(endpoint caller) {

    }
}
```

New syntax:

```ballerina
service<http:Service> update_token bind { port: 9295 } {

    # Updates the access token.
    #
    # + caller - Endpoint
    @http:ResourceConfig {
        methods:["GET"]
    }
    one_px_image(endpoint caller) {

    }
}
```

### Changes to record/object field syntax

Old syntax:

```ballerina
type foo record {
    string a,
    string b,
};

type bar object {
    string a,
    string b,
};
```

New syntax:

```ballerina
type foo record {
    string a;
    string b;
};

type bar object {
    string a;
    string b;
};
```

### New anonymous function syntax

The syntax of the anonymous function has been changed to resemble the normal function definition.

Old syntax:

```ballerina
var lambda = (int b) => (int) {
    int x = b * b;
    return x;
};
```

New syntax:

```ballerina
var lambda = function (int b) returns (int) {
    int x = b * b;
    return x;
};
```

A new “Arrow Function Expression” is introduced as a simple alternative to anonymous functions that have only a return statement in the body block. Syntax is as follows:

function (int, string) returns string lambdaVar = (param1, param2) => param2.toUpper();

## Runtime changes

### Removal of implicit cast from int to float

Now it is necessary to explicitly define a float with a decimal point.

```ballerina
float x = 0; // Compile Error
float x = 0.0; // Working Code
```

### Changes to the `main` function

- Based on the latest changes, the `main` function has to be marked `public` and can return an int.
- Now it is also possible to execute `ballerina run` to invoke any `public` function in the entry package. For example, if you want to invoke the public function `add` in the `calculator` package, you can execute `ballerina run` and specify the integer arguments `4` and `5` as follows:
  - ballerina run calculator:add 4 5
  - If a function is not specified, `main` is considered as the function to run.
- The function invoked via `ballerina run` (including the `main` function) will be data-binding and can have zero or more parameters of any supported type, including any number of required/defaultable parameters and a single rest parameter. This function can also return a value. For example, consider the following public function in the `currency` package:
    ```ballerina
  function queryChanges(string host, int port = 8080, string… params) returns float {
  }
    ```
    invoking this function via `ballerina run`
    ballerina run --printreturn currency:queryChanges localhost -port=8181 high day
    will result in the following value assignments

- host ← “localhost”
- port ← 8181
- params ← [“high”, “day”]
    When the `--printreturn` flag is set, the return value is printed to the standard out stream.

### Option/param order enforcement with the run command

Option vs parameter ordering is enforced with the run command, and all options need to be specified before the file/package to run. 
Any arguments (even if they match an option recognized by the run command) specified after the file/package are now considered program arguments.
A config file can be specified with the run command as follows:

```ballerina
ballerina run --config myConfig.conf calculator 4 5
```

Specifying the `--config` argument as follows does not identify it as an option:

```ballerina
ballerina run calculator --config myConfig.conf 4 5
```

# Ballerina Native
Ballerina Native feature is a LLVM based backend for the Ballerina compiler. It allows Ballerina programs to be compiled into native executables.

## Prerequisites

Unix based (Linux/MacOS) operating system to run the initial version. Other operating systems will be supported in future versions.
Be sure you have the GCC compiler installed. (It is possible to use `clang` or `ld`. But the current recommended option is `gcc`)

## Supported language constructs

- Main function (string args are ignored)
- Int and boolean types
- If condition
- while loop
- Function calls and return values
- Partial support - println for int values

## How to run

build a Ballerina program with the native flag
`ballerina build --native -o myballerinamain  myballerinamain.bal`
Run the created executable
`./myballerinamain`

## Command line flags

Following additional flags are valid when the --native flag is provided:
`--dump-bir` prints the Ballerina intermediate representation
`--dump-llvm-ir` prints the LLVM intermediate representation assembly code

# Improvements

## Language & Runtime

- Tracking tainted state changes of function parameters : Taint analyzer keeps track of the tainted state of the parameters in different execution conditions. This information is used to update the tainted state of the arguments used in a function invocation, and to make sure that the tainted state of arguments propagate correctly when the parameter is an out parameter or an in-out parameter.
- Introduction of abstract objects. An abstract object is identified by the ‘abstract’ keyword. It describes the type of each field and each method, but not the implementation of the methods. An abstract object type must not have an object constructor method and does not have an implicit initial value. An abstract object type cannot be initialized using the object initializer.
    ```ballerina
  public type Foo abstract object {
      public string name;
      public int id;

      function getName() returns string;

      function getID() returns int;
  };
    ```
- Introduction of record iteration support. Records are now an iterable type. Therefore, the foreach loop and iterable operations can now be used with records. When iterating a record, you can either iterate over the fields (i.e., field name and value pair) or iterate only over the field values.

    ```ballerina
  type Person record {
       string name;
       int age;
       string address;
  };
    ```
    Foreach loop can be used on an instance of this record as follows:
    ```ballerina
  foreach field, value in person {
       io:println(field + " : " + <string>value);
  }
    ```
    Or if iterating only through the field values:
    ```ballerina
  foreach value in person {
       io:println(value);
  }
    ```

- Introduction of Channel type. Channel is a constrained type that can be defined only as a top level node. It is introduced for communication between parallel processes in Ballerina programs. Channels can be used for message correlation by sending and receiving messages via different resources to the same channel. It can also be used for  inter-worker communication and synchronize workers.

Defines a channel with json constrained type

```ballerina
channel<json> jsonChannel;
```

Send a message to the channel

```ballerina
    # One of the receivers waiting on this key receives it.
    # If there is no receiver, the message is stored and execution continues.
    # A receiver can arrive later and fetch the message.
    message -> jsonChannel, key;
```

Receive a message from the channel with given key. Execution waits here if the message is not available

```ballerina
json result <- jsonChannel, key;
```

## Build & Package Management

- Build mandated in `ballerina push` and `install`: By default the source is built before pushing the package to Ballerina Central and before installing the package to the home repository. Use the `--no-build` flag to skip building before pushing or installing.
    `ballerina push <package-name> --no-build`
    `ballerina install <package-name> --no-build`
- Introduction of the `uninstall` command : Allows uninstalling packages that are installed to the home repository and shared across other projects. The uninstall command can take the following options: 
    `ballerina uninstall <org-name>/<package-name>:<version>`

## Standard Library

- Support for HTTP 1.1 pipelining.
- Enhanced support for compression/decompression.
- Change of statusCode and reason to be optional parameters for the close action of WebSocket endpoint.
- Support to directly configure SSL certificates and keys without keystores.
- Support to define enum type in gRPC request/response messages.

## IDE Plugins

- Diagram editing support in VSCode.
- Language server integration support in IDEA plugin: This results in providing all language intelligence support through the language server.
- Variable definition auto generation code action.
- Attached function’s implementation snippe
- Object constructor auto generation code action.

## Extensions

### Kubernetes

Helm chart generation support

# Performance results

From this release onwards Ballerina [performace test results](https://github.com/ballerina-platform/ballerina-lang/blob/v0.982.0/performance/benchmarks/summary.md) will be available in the repository as a summary.

# Bug Fixes

Please refer [Github milestone](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.982.0+is%3Aclosed+label%3AType%2FBug) to view bug fixes

# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour] (https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).