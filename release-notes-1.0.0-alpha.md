# Overview of Ballerina 1.0.0 Alpha
Ballerina 1.0.0 Alpha is here! It is a massive milestone before our official 1.0.0 release, so we want as many of you to try it out and give us feedback via our [Slack channel](https://ballerina-platform.slack.com/), [Google Group](https://groups.google.com/forum/#!forum/ballerina-dev) or [Github](https://github.com/ballerina-platform/ballerina-lang).
Ballerina 1.0.0 Alpha consists of improvements to the language syntax and semantics based on the stable language specification version 2019R2 and new features and enhancements to the standard library modules, extensions, and tooling.

# Highlights
- Based on a stable language specification: 2019R2
- Introduces a brand new Ballerina compiler back-end that targets the JVM
- Significant performance improvements over the previous Ballerina runtime (BVM)
- Java interoperability (allows you to call Java code from Ballerina)

# Breaking Changes

## Breaking Language Changes

### Builtin library

The `builtin` module has been removed. Some of the functionalities provided by the `builtin` library is now provided  by the newly-added `lang` library.

- The `freeze()` builtin method has been replaced with the `cloneReadOnly()` lang library function. `cloneReadOnly()` can be called only on variables of the type `anydata`. It creates and returns a clone of the value that has been made immutable (for non-simple basic types).

Previous Syntax

```ballerina
map<string> m2 = m.freeze();  
```

New Syntax

```ballerina
map<string> m2 = m.cloneReadOnly();
```

- The `convert()` builtin method has been replaced with the `constructFrom()` lang library function. `constructFrom()` can only be called on a type descriptor `T` where the `T` is a subtype of `anydata`. It accepts an `anydata`  value as an argument and returns a value of the type `T` constructed using a deep copy of the provided argument. If the construction fails, it returns an error.

Previous Syntax

```ballerina
json j = { name : "tom", age: 2};
Person|error p = Person.convert(j);
```

New Syntax

```ballerina
json j = { name : "tom", age: 2};
Person|error p = Person.constructFrom(j);
```

- The following behaviours, which were previously associated with the `convert()` method is now provided by the lang library functions of the relevant types.
- Previously, `convert()` was used to parse string literals. Now, the `lang.int`, `lang.float`, and `lang.decimal` modules have a `fromString()` function, which accepts a string literal and parses it.

Previous Syntax

```ballerina
int|error x = int.convert(“100”);
```

New Syntax

```ballerina
import ballerina/lang.’int; // Need to import `lang.int`

int x = ‘int:fromString(“100”);
```

- Previously, when invoked on the `string` typedesc, `convert()` returned a string representation of the value. Now, the `lang.value` module provides a `toString()` function, which returns a human-readable string representation of a value.

Previous Syntax

```ballerina
json person = {“name”:”John”, “age”:25};
string|error str = string.convert(person);
```

New Syntax

```ballerina
json person = {“name”:”John”, “age”:25};
string str = person.toString();
```

- The `stamp()` method has been removed.

### Maps and Records

- The semantics of the `{`, `}` and `{|`, `|}` delimiters have changed. A record type descriptor written using the `{|` and `|}` delimiters defines a record type, which only accepts mapping values with the same fields as the ones described. A record type descriptor written using the `{` and `}` delimiters define a record type, which additionally allows pure type fields apart from the described fields, i.e., `record {}` is equivalent to `record {| anydata…; |}`. 

Previous Syntax

```ballerina
// Open record with a field `a`. It additionally allows
// `anydata|error` fields.
type Foo record {
   string a;
};

// Open record with a field `a`. It additionally allows
// `int` fields.
type Bar record {
   string a;
   int...;
};

// Closed record, which only allows a `string` field named `a`.
type Baz record {|
   string a;
|};

```

New Syntax

```ballerina
// Open record with a field `a`. It additionally allows
// `anydata` fields.
type Foo record {
  string a;
};

// Open record with a field `a`. It additionally allows
// only `int` fields.
type Bar record {|
  string a;
  int...;
|};

// Closed record, which only allows a `string` field named `a`.
type Baz record {|
  string a;
|};
```

- The default `record rest` field type has been changed to `anydata` from `anydata|error`.
- The syntax to specify expressions as keys in the mapping constructor has changed. Now, expressions need to be enclosed in `[]` (e.g., `[expr]`).

Previous Syntax

```ballerina
map<string> m = { getString(): "value" };
```

New Syntax

```ballerina
map<string> m = { [getString()]: "value" };
```

- String literals can now be used as keys in the mapping constructor for a record. The key for a `rest` field should be either a string literal or an expression in the mapping constructor (i.e., cannot be an identifier). 

Previous Syntax

```ballerina
type Foo record {
   string bar;
   int...;
};

Foo f = { bar: "test string", qux: 1 }; // `qux` is a rest field
```

New Syntax

```ballerina
type Foo record {|
   string bar;
   int...;
|};

Foo f = { bar: "test string", "qux": 1 }; // `qux` is a rest field
```

- Mapping values are now iterable as sequences of their members (values) instead of sequences of key-value pairs. A lang library function `entries()` is available to retrieve an array of key-value pairs for a mapping value.

Previous Syntax

```ballerina
foreach (string, int) (k, v) in m {
   io:println("Key:   ", k);
   io:println("Value: ", v);
}
```

New Syntax

```ballerina
// Iterating values.
foreach int v in m {
   io:println("Value: ", v);
}

// Iterating entries.
foreach [string, int] [k, v] in m.entries() {
   io:println("Key:   ", k);
   io:println("Value: ", v);
}
```

### Arrays and Tuples

- The requirement for array element types to have an implicit initial value to allow declaring variable-length arrays has been removed. Instead, when a value is being added to the array at runtime, the index is greater than the length of the list, and the element type does not have a filler value, it would result in a panic.  

Previous Syntax

```ballerina
(int|string)[] arr = []; // Fails at compile time.
arr[1] = 1;
```

New Syntax

```ballerina
(int|string)[] arr = [];
arr[1] = 1; // Fails at runtime.
```

- Tuple types now use brackets instead of parentheses. 

Previous Syntax

```ballerina
(int, string) t = (1, "hello world");
```

New Syntax

```ballerina
[int, string] t = [1, "hello world"];
```

- Tuple types now support rest descriptors. Therefore, the following syntax is valid now.

```ballerina
[int, string, boolean...] t = [1, "hello world", true, true];
[int...] t3 = [1, 2];
```

### Objects

- Objects outside method definitions are no longer allowed. All object function definitions need to be specified within the object itself. The following syntax is invalid now.

```ballerina
type Foo object {
   int code = 0;

   function printCode();
};

function Foo.printCode() {
   // print code
}
```

### Functions and Methods

- Arguments for required parameters of a function can now also be passed as named arguments, while arguments for defaultable parameters can be passed as positional arguments. To avoid ambiguity, all named arguments need to be specified after the positional arguments.
- Parameters of a function can now be marked `public` and only arguments can be passed by the name to such parameters when invoking a function from an imported module. These arguments can still be passed as positional arguments.

### Error Type and Constructor

- The error detail type must now belong to the detail type defined in the error lang library.

```ballerina
public type Detail record {|
   string message?;
   error cause?;
   (anydata|error)...;
|};
```

- The error constructor now accepts detail fields as individual named arguments as opposed to accepting a single mapping as the `detail` argument.	 
Previous Syntax

```ballerina
Detail detail = { message: "error message", code: 1100 };
error e = error("error reason", detail);
```

New Syntax

```ballerina
error e = error("error reason", message = "error message", code = 1100);
```

### Annotations

- Annotation declaration syntax and attachment points have been revised. Annotations can now be declared to be available only at compile time (source only annotations) or at both compile-time and runtime.

Previous Syntax

```ballerina
annotation<service> annot Foo;
```

New Syntax

```ballerina
annotation Foo annot on service;
```

### Expressions

- Integer range expressions now return objects belonging to the iterable abstract object type instead of lists (arrays).
- Field access for records, objects, and JSON has changed. Field access can be used to access the fields of an object or required fields of a record. Field access on a lax-typed variable (`json|map<json>`) now returns `json|error`. The field access operator does not lift nil now. 
- Member access is allowed with lists and subtypes of optional mappings. Nil lifting is supported in the latter case. 
- The error lifting operator (`!`) has been removed. Error lifting now happens only with field or optional field access for lax types.
- Calls with `start` are now considered as actions. As a result, they are not allowed within expressions.
- Delimited identifier syntax has been changed.

```ballerina
string ^"1" = "identifier one";
string ^"identifier two" = "identifier two";
```

New Syntax

```ballerina
string '1 = "identifier one";
string 'identifier\ two = "identifier two";
```

- The `untaint` unary operator has been replaced by an annotation to mark a value as trusted.	 

Previous Syntax

```ballerina
var untaintedValue = untaint taintedValue;
```

New Syntax

```ballerina
var untaintedValue = <@untainted> taintedValue;
```

# What's new in Ballerina 1.0.0-alpha

## Language

- A set of modules, which contain the functions associated with the basic types. These modules are referred to as the lang library. Each basic type has a corresponding lang library module. Additionally, there is also the `lang.value` module, which holds functions common for all the types. The following is the list of the lang library modules.

  - `ballerina/lang.value`
  - `ballerina/lang.array` for basic type list
  - `ballerina/lang.decimal` for basic type `decimal`
  - `ballerina/lang.error` for basic type `error`
  - `ballerina/lang.float` for basic type `float`
  - `ballerina/lang.future` for basic type `future`
  - `ballerina/lang.int` for basic type `int`
  - `ballerina/lang.map` for basic type `mapping`
  - `ballerina/lang.object` for basic type `object`
  - `ballerina/lang.stream` for basic type `stream`
  - `ballerina/lang.string` for basic type `string`
  - `ballerina/lang.table` for basic type `table`
  - `ballerina/lang.typedesc` for basic type `typedesc`
  - `ballerina/lang.xml` for basic type `xml`

- The basic type `handle` has been added. A `handle` value is a reference to storage of a Ballerina program that is managed externally. `Handle` values are useful only in conjunction with functions that have external function bodies; in particular, a new handle value can be created only by a function with an external function body.

- The error reason is now optional if the reason can be inferred based on the contextually expected type.

```ballerina
type Detail record {
   int code;
};

const FOO = "foo";

type FooError error<FOO, Detail>;

FooError e1 = error(FOO, code = 3456);
FooError e2 = error(code = 3456); // Also valid now, reason is set as "foo"
```

- A unary operator `typeof` has been introduced to retrieve a typedesc value for the runtime type of a value.

```ballerina
typedesc t = typeof valueExpr;
```

- A binary operator `.@` has been introduced to access annotation values at runtime.	 

```ballerina
annotation Foo annot on service;
typedesc t = typeof serviceValue;
Foo? fooAnnot = t.@annot;
```

- Expressions are now allowed as default values for function parameters.

- The concept of lax typing has been introduced allowing less stricter static typing for types identified as lax. With lax typing, some of the static typing checks are moved to the runtime,resulting in error returns at runtime instead. With this release, `json` and `map<T>` where `T` is lax are considered as lax. 
- An optional field access operator `?.` has been introduced to access possibly undefined mapping members. Optional field access on lax types may return `error`, if applied to a non-mapping value.

## Runtime

We are introducing a brand new implementation (jBallerina) of the Ballerina language spec that targets the JVM. The jBallerina compiler produces an executable JAR file for a Ballerina program by directly transforming Ballerina sources to Java bytecode. With jBallerina, we are deprecating and removing our previous Ballerina runtime implementation (BVM). The jBallerina comes with significant performance improvements over the BVM.

### Java Interoperability

Java interoperability is a key feature in jBallerina that allows you to call Java code from Ballerina. It also enables you to embrace the capabilities of Ballerina for new projects while utilizing existing Java libraries that you or your company invested in for years.

## Project Structure & Build Tools

- Ballerina project structure should match the following.

```
project-name/
- Ballerina.toml
- src/
-- mymodule/
--- Module.md  	<- module-level documentation
--- main.bal   	<- Contains the default main method.
--- resources/ 	<- resources for the module (available at runtime)
--- tests/     	<- tests for this module (e.g. unit tests)
---- testmain.bal  <- test file for main
---- resources/	<- resources for these tests
- tests/       	<- integration tests
-- integration.bal <- integration test file
-- resources/  	<- integration test resources
- target/     	<- directory for compile/build output
-- bin/       	<- Executables will be created here
-- balo/      	<- .balo files one per built module
--- mymodule.balo  <- balo object of module1
-- cache      	<- BIR, JAR cache directory

```

- To push to staging central, set the following env variable with the alpha release. (https://staging-central.ballerina.io)

```ballerina
export BALLERINA_DEV_STAGE_CENTRAL=true
```

- To create a new project with a hello world, use the *new* command. This initializsa a new directory.
```$ ballerina new <project-name>```

- To create a module, use the *create* command inside the project.
```$ ballerina create <modulename> [-t main|service]```

- If you are building a library, use the *compile* command. This generates a BALO to push to central.
```$ ballerina compile```

- To create an executable, use the *build* command.
```$ ballerina build```

- To run the executable, use the *run* command.
```$ ballerina run mymodule-executable.jar```

### Ballerina Central

- Supports pushing of Ballerina modules with native Java libraries.

## Standard Library

- Revamped the NATS connector to support both NATS and Streaming Servers.
- Introduce the StdLib module-wise errors as a replacement for the builtin error. 
  E.g., Ballerina HTTP Error types include http:ClientError, http:ListenerError, http:ClientAuthError etc.
- Introduce capability to engage custom providers and handlers for inbound/outbound authentication
- Introduce OAuth2 inbound authentication
- Introduce own modules for different authentication mechanisms (JWT, LDAP, OAuth2 etc.)
- Improve LDAP APIs by decoupling usage with an auth provider
- Introduce support for consumer services with data binding, queue-groups, different starting position types etc.
- Introduce prior knowledge support to the HTTP/2 client
- Add flow control support to HTTP/2 client and server
- Data binding support for RabbitMQ connector. The supported types include `int`, `float`, `string`, `json`, `xml`, `byte[]`, and `records`.
- Transaction support in RabbitMQ broker and added Ballerina local transaction support for the module. Ballerina RabbitMQ local transactions follow the RabbitMQ  broker semantics transaction model.
- Introduce XSL transformation support
- Introduce "system" APIs to perform system-bound file operations such as create file, create directory, move directory, rename file, get file metadata, copy file etc.
- H2 and MySQL database client modules have been discontinued. The JDBC client module can be used to interact with relational databases.

## IDEs & Language Server

### IntelliJ IDEA Plugin

- Introduce Ballerina home auto detection capability.
- Introduce Ballerina sequence diagram view.
- Revamp the debugger using a DAP (Debugger Adapter Protocol) client.
- Introduce in-place renaming support.
- Add language-server-based signature help.

### Tooling

- Ballerina Formatter: Ballerina source formatting CLI tool.
- OpenAPI to Ballerina generator CLI tool.
- Ballerina to OpenAPI generator CLI tool.
- OpenAPI validator compiler plugin.
- Introduce Debug Adapter Protocol implementation.
