# Overview of JBallerina 1.1.0

Ballerina 1.1.0 is the year end release which has significant improvements  to the developer tooling, standard library modules, compiler, and runtime.

# Highlights

* Ballerina tool
* Improved runtime performance
* Improved support for Java interoperability
* Optimized compiler for better IDE experience
* Based on a stable language specification: 2019R3
* Includes a number of important bug fixes.

# What's new in Ballerina 1.1.0

## Ballerina tool

Ballerina 1.1.0 introduces ballerina tool with set of commands which allows users to manage Ballerina distributions. These are functionalities allowed by the tool.

* Update current distribution to the latest patch version
  * ``ballerina dist update``
* Fetch a specified distribution and set it as the active version
  * ``ballerina dist pull``
* Set a specified distribution as the active distribution
  * ``ballerina dist use``
* List locally and remotely available distributions
  * ``ballerina dist list``
* Remove a specified distribution
  * ``ballerina dist remove``

Apart from that in order update tool itself following command could be used
ballerina update

## Language

The language implementation is based on the stable language specification version 2019R3. The implementation has been improved and stabilized by fixing critical specification deviations and bugs in the 1.0.0 release. Some of these changes are backward incompatible. Check the "Language changes since JBallerina 1.0.0" section for the full list of changes.

The parser used by the compiler has been improved with a file-based cache. This significantly improves the performance of the JBallerina Language Server, which in turn improves the user experience of the IDEs. Additionally, few minor fixes have been done to Compiler AST to support better IDE insights.

## Runtime

* Runtime performance improvements
* Improved runtime type checking performance
* Improved performance for creating and accessing maps, records and arrays.
* Java interoperability improvements
* Supports using `string` type, finite types, and function pointers, for parameters and/or return type in java interop functions.
* Improved compile time validation for interop method signatures.

## Project Structure & Build Tools

## Standard Library

* Path param suffix support for HTTP service dispatcher
* HTTP cookie support
* 100-continue support for HTTP2
* Add observability support for gRPC
* Add observability support for WebSocket
* Add observability support for NATS connector
* Add support for publishing messages with a replyTo callback service in NATS

## IDE Plugins & Language Server

* IntelliJ live template support for ballerina code snippets.
* Go-to definition support for both VSCode & IntelliJ plugins 
* Add CLI commands to start Language Server and start Debug Adaptor
* Diagram support for tests
* Improvements to language support
* Performance improvements to language server
* Significant improvements to decrease the response time for language intelligence features such as auto completion, diagnostics and etc.

# Dev Tools

### API Documentation generator

* Fix type label generation for array types
* Prevent link generation for non-public types

### Open API tool

* Support for oneOf & allOf schema types
* Improvements to Ballerina record types generator

# Detailed list of changes from 1.0.0 to 1.1.0

## Language changes since JBallerina 1.0.0

This section highlights key Language changes since JBallerina 1.0.0. These changes are backward-incompatible and the 1.1.0 compiler generates compile-time errors now.  

You can find the list of Language and Compiler related fixes in this release from [here](https://github.com/ballerina-platform/ballerina-lang/issues?page=2&q=is%3Aissue+milestone%3A%22Ballerina+1.1.0%22+is%3Aclosed+label%3AArea%2FLanguage&utf8=%E2%9C%93).

### String Literal

The JBallerina 1.0.0 compiler allowed `0xA` and `0xD` code points in string literals even though they are invalid in a string literal. Validation has been added to disallow the same in this release.

```ballerina
// Following is a compile time error.
string s = "hello
world";
```

### Foreach Statement

The compiler now validates the if the correct type is specified for record iteration. The following code should produce a compilation error now. 

```ballerina
function testForeachRecord() {
    record {| string s; anydata|error...; |} value = {s : "aString"};
    foreach any item in value { // Now gives a compilation error.
      // Some Code
    }
}
```

### While Statement

This release includes several bug fixes that improve the detection of potentially uninitialized variables inside a while loop. This may break source code compatibility with the 1.0.0 release.

### Destructuring assignment statement

The [Implicit variable type narrowing](https://ballerina.io/spec/lang/2019R3/#section_7.11) section of the Ballerina language specification says “the narrowed type for a variable x no longer applies as soon as there is a possibility of x being assigned to”. But this validation was not enforced properly when used in the context of a [Destructuring assignment statement](https://ballerina.io/spec/lang/2019R3/#section_7.14.4. Furthermore, final variable analysis was also incomplete with destructuring assignments. These validations are corrected with this release.

### Check Expression

The 1.0.0 compiler does not properly validate the compatibility of the check expression’s error type with the enclosing function’s return type. This flaw leads to programs panicking at runtime when the expression evaluates to an incompatible error type. The 1.1.0 compiler introduces this compile-time validation. 

The `check` expression acts as a conditional return statement. This fact was ignored in taint analysis and within the transaction block, but has been fixed with this release. Now the `check` expression enforces marking function return values as `@tainted` where applicable and  `check` expressions are not allowed within a transaction block similar to the return statement.

### Function Type and Value

Function type descriptors and anonymous functions now support rest parameters.

Example:

```ballerina
var foo = function (string b, int... c) returns int {
        return 5;
    };
function (string, int...) returns int bar = foo;
```

Additionally, the 1.0.0 compiler accepted arrays in place of functions pointer rest arguments. It has now been fixed to only accept rest arguments.

```ballerina
function myFuncton(string b, int... c) { }

public function main() {
    function(string, int...) func1 = myFuncton;

    // Worked in 1.0.0, now a compile time error.
    func1("a", [1, 2, 3]); // Error

    // Compile time error in 1.0.0, but correct behaviour
    func1("a", ...[1, 2, 3]); // or func1("a", 1, 2, 3);

    function (string a, int[] b) func2 = myFuncton; // Worked in 1.0.0, now a compile time error.
    function (string a, int... b) func3 = myFuncton; // Not supported in 1.0.0, but correct behavior.
}
```

### Object Type

[Object Initialization](https://ballerina.io/spec/lang/2019R3/#section_5.3.2.5) specifies that if at any point in the `__init` method there are any potentially uninitialized fields, then using the `self` variable other than to access or modify the value of a field should produce a compile-time error. This validation has been added to the compiler in the 1.1.0 release. The following code should produce a compilation error now.

```ballerina
type Person object {
    string name;
    int age = 0;

    public function __init(string name, int age) {
        self.setAge(age); // error: field `name` is not initialized
        self.name = name;
    }

    function setAge(int age) {
      self.age = age;
    }
};
```

When the `__init` method is not present in an object, the 1.0.0 compiler skipped validating the arguments to the `new` expression. This behavior is corrected with the 1.1.0 release.

### Iterable Objects

This release adds support for [Iterable objects](https://ballerina.io/spec/lang/2019R3/#section_5.5.2). Moreover the result of a range expression has also been updated to be a new object belonging to the abstract object type `Iterable<int>`.

```ballerina
import ballerina/io;

type IterableObject object {
    function __iterator() returns IntIterator {
        return new IntIterator();
    }
};

type IntIterator object {
    int i = 0;
    public function next() returns record {|int value;|}? {
        self.i += 1;
        return self.i < 10 ? {value: self.i} : ();
    }
};

public function main() {
    foreach int i in new IterableObject() {
        io:println(i);
    }
}
```

### Worker Actions

The 2019R3 specification defines that in a [single receive action](https://ballerina.io/spec/lang/2019R3/#section_7.8.2.1) if the sending worker terminated with failure, the evaluation of the single receive completes normally with the result being the termination value of the sending worker, which will be an error. This validation was not enforced for the sync send action, but has been added with this release.

The following results in a compilation error now.

```ballerina
function WorkerInteraction () {
    worker w1 returns error? {
        int i = 10;
        if (true) {
            return error("Error!");
        }
        i ->> w2;
    }

    worker w2 {
        int i = <- w1; // error: expected type int|error
    }
}
```

### Documentation String

[Ballerina Flavored Markdown](https://ballerina.io/spec/lang/2019R3/#section_9.3) syntax provides conventions for referring Ballerina-defined names from within the documentation string in a source file. This support has been added in JBallerina 1.1.0.

Example:

```ballerina
    # Adds parameter `x` and parameter `z`
    # + x - one thing to be added
    # + y - another thing to be added
    # + return - the sum of them
    function add (int x, int y) returns int { return x + y; }
```

In this example ``parameter `x` `` and  ``parameter `z` `` are validated by the compiler.  The compiler generates a warning message for  ``parameter `z` `` since there is no parameter with the name `z`.

### Taint Analysis

This release adds validations to analyze the taintedness of listeners when declared inline with the service declaration. This may break source code compatibility with the 1.0.0 release.
