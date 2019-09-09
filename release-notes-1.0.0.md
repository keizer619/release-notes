# Overview of Ballerina 1.0.0

Ballerina 1.0.0 is here! We would like you to try it out and give us feedback via our [Slack channel](https://ballerina-platform.slack.com/), [Google Group](https://groups.google.com/forum/#!forum/ballerina-dev), [Twitter](https://twitter.com/ballerinalang), or [Github](https://github.com/ballerina-platform/ballerina-lang).
Ballerina 1.0.0 consists of improvements to the language syntax and semantics based on the stable language specification version 2019R3 and new features and enhancements to the standard library modules and developer tooling.

# Highlights

- Based on a stable language specification: 2019R3
- Introduces a brand new Ballerina compiler back-end that targets the JVM
- Significant performance improvements over the previous Ballerina runtime (BVM)
- Java interoperability (allows you to call Java code from Ballerina)
- Major redesign of Ballerina developer tools

# What's new in Ballerina 1.0.0

## Language

- A set of modules, which contain functions associated with the basic types have been introduced. Collectively, these modules are referred to as the lang library. Each basic type has a corresponding lang library module. Additionally, there is also the `lang.value` module, which holds functions common for all the types. The following is the list of lang library modules.

  - `ballerina/lang.value`
  - `ballerina/lang.array` for list types
  - `ballerina/lang.decimal` for basic type `decimal`
  - `ballerina/lang.error` for basic type `error`
  - `ballerina/lang.float` for basic type `float`
  - `ballerina/lang.future` for basic type `future`
  - `ballerina/lang.int` for basic type `int`
  - `ballerina/lang.map` for mapping types
  - `ballerina/lang.object` for basic type `object`
  - `ballerina/lang.stream` for basic type `stream`
  - `ballerina/lang.string` for basic type `string`
  - `ballerina/lang.table` for basic type `table`
  - `ballerina/lang.typedesc` for basic type `typedesc`
  - `ballerina/lang.xml` for basic type `xml`

- The basic type `handle` has been added. A `handle` value is a reference to storage area of a Ballerina program that is managed externally. `Handle` values are useful only in conjunction with functions that have external function bodies; in particular, a new handle value can be created only by a function with an external function body.

- The error reason is now optional if the reason can be inferred based on the contextually-expected type.

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

- The concept of lax typing has been introduced allowing less stricter static typing for types identified as lax. With lax typing, some of the static typing checks are moved to the runtime returning errors at runtime instead. With this release, `json` and `map<T>` where `T` is lax are considered as lax.
- An optional field access operator `?.` has been introduced to access possibly-undefined mapping members. Optional field access on lax types may return `error` if applied to a non-mapping value.

## Runtime

This release introduces a brand new implementation (jBallerina) of the Ballerina language spec, which targets the JVM. The jBallerina compiler produces an executable JAR file for a Ballerina program by directly transforming Ballerina sources to Java bytecode. With jBallerina, the previous Ballerina runtime implementation (BVM) will be deprecated and removed. jBallerina comes with significant performance improvements over the BVM.

### Java Interoperability

Java interoperability is a key feature in jBallerina that allows you to call Java code from Ballerina. It also enables you to embrace the capabilities of Ballerina for new projects while utilizing existing Java libraries that you or your organization invested in for years.

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
   ---- main_test.bal  <- test file for main
   ---- resources/	<- resources for these tests
   - target/     	<- directory for compile/build output
   -- bin/       	<- Executables will be created here
   -- balo/      	<- .balo files one per built module
   --- mymodule.balo  <- balo object of module1
   -- caches/      	<- BIR, JAR cache directory

   ```

- To create a new project with a hello world, use the *new* command. This initializes a new directory.
   ```
   $ ballerina new <project-name>
   ```

- To add a module, use the *add* command inside the project.
   ```
   $ ballerina add <modulename> [-t main|service]
   ```

- To create an executable, use the *build* command.
   ```
   $ ballerina build
   ```

- To run the executable, use the *run* command.
   ```
   $ ballerina run mymodule.jar
   ```

### Ballerina Central

- Supports pushing of Ballerina modules with embedded, dependent native Java libraries.

## Standard Library

- Revamp the NATS connector to support both NATS and Streaming Servers.
- Introduce the standard library module-wise errors as a replacement for the builtin `error`.
  e.g., Ballerina HTTP Error types include `http:ClientError`, `http:ListenerError`, `http:ClientAuthError` etc.
- Introduce capability to engage custom providers and handlers for inbound/outbound authentication.
- Introduce OAuth2 inbound authentication.
- Introduce own modules for different authentication mechanisms (JWT, LDAP, OAuth2 etc.).
- Introduce prior knowledge support to the HTTP/2 client.
- Add flow control support to HTTP/2 client and server.
- Introduce XSLT transformation support.
- `ballerina/h2` and `ballerina/mysql` database client modules and the `ballerina/sql` module have been discontinued. The `ballerinax/java.jdbc` client module can be used to interact with relational databases.
- The byte channel read API was updated to return only `byte[]|io:Error`.
- Introduce out of the box support for messaging with Kafka.
- RabbitMQ, JMS, Artemis, WebSub and LDAP modules are available through Ballerina Central.
- APIs for performing file system operations such as creating files, creating directories, moving directories, renaming files, fetching file metadata, copying files etc. are now available through the `ballerina/file` module.
- Most of the APIs of the `ballerina/encoding` module were removed since they are now supported via the lang library.
- Three new utility modules were introduced to manipulate built-in `string`, `json` and `xml` types.

## IDEs & Language Server

### IntelliJ IDEA Plugin

- Introduce Ballerina home auto detection capability.
- Introduce Ballerina sequence diagram view.
- Revamp the debugger using a DAP (Debugger Adapter Protocol) client.

### Tooling

- Ballerina Formatter: Ballerina source formatting CLI tool.
- OpenAPI to Ballerina generator CLI tool.
- Ballerina to OpenAPI generator CLI tool.
- OpenAPI validator compiler plugin.
- Introduce Debug Adapter Protocol implementation.

# Breaking Changes from 0.991.0

## Breaking Language Changes

### Builtin library

The `ballerina/builtin` module has been removed. Some of the functionalities provided by the `ballerina/builtin` library is now provided by the newly-added lang library.

- The `freeze()` builtin method has been replaced with the `cloneReadOnly()` lang library function. `cloneReadOnly()` can be called only on variables of the type `anydata`. It creates and returns a clone of the value that has been made immutable (for non-simple basic types).

   Previous Syntax

   ```ballerina
   map<string> m2 = m.freeze();  
   ```

   New Syntax

   ```ballerina
   map<string> m2 = m.cloneReadOnly();
   ```

- The `convert()` builtin method has been replaced with the `constructFrom()` lang library function. `constructFrom()` can only be called on a type descriptor `T` where the `T` is a subtype of `anydata`. It accepts an `anydata` value as an argument and returns a value of the type `T` constructed using a deep copy of the provided argument. If the construction fails, it returns an error.

   Previous Syntax

   ```ballerina
   json j = {name:"tom", age:2};
   Person|error p = Person.convert(j);
   ```

   New Syntax

   ```ballerina
   json j = {name:"tom", age:2};
   Person|error p = Person.constructFrom(j);
   ```
- The following behaviours, which were previously associated with the `convert()` method is now provided by the lang library functions of the relevant types.
   - Previously, `convert()` was used to parse string literals. Now, the `lang.int`, `lang.float`, and `lang.decimal` modules have a `fromString()` function, which accepts a string literal and parses it.

      Previous Syntax

      ```ballerina
      int|error x = int.convert("100");
      ```

      New Syntax

      ```ballerina
      import ballerina/lang.'int; // Need to import `lang.int`

      int x = 'int:fromString("100");
      ```

   - Previously, when invoked on the `string` type descriptor, `convert()` returned a string representation of the value. Now, the `lang.value` module provides a `toString()` function, which returns a human-readable string representation of a value.

      Previous Syntax

      ```ballerina
      json person = {"name":"John", "age":25};
      string|error str = string.convert(person);
      ```

      New Syntax

      ```ballerina
      json person = {"name":"John", "age":25};
      string str = person.toString();
      ```

- The `stamp()` method has been removed.

### Maps and Records

- The semantics of the `{`, `}` and `{|`, `|}` delimiters have changed. A record type descriptor written using the `{|` and `|}` delimiters defines a closed record type, which only accepts mapping values with the same fields as the ones described. A record type descriptor written using the `{` and `}` delimiters define a record type, which additionally allows pure type fields apart from the described fields, i.e., `record {}` is equivalent to `record {| anydata…; |}`. 

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

- The default record rest field type has been changed to `anydata` from `anydata|error`.
- The syntax to specify expressions as keys in the mapping constructor has changed. Now, expressions need to be enclosed in `[]` (e.g., `[expr]`).

   Previous Syntax

   ```ballerina
   map<string> m = { getString(): "value" };
   ```

   New Syntax

   ```ballerina
   map<string> m = { [getString()]: "value" };
   ```
- String literals can now be used as keys in the mapping constructor for a record. The key for a `rest` field should either be a string literal or an expression in the mapping constructor (i.e., cannot be an identifier). 

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

- Outside method definitions are no longer allowed for objects. All object function definitions need to be specified within the object itself. The following syntax is invalid now.

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

- Now, arguments for required parameters of a function can also be passed as named arguments, while arguments for defaultable parameters can be passed as positional arguments. To avoid ambiguities, all named arguments need to be specified after the positional arguments.
- Parameters of a function can now be marked `public` and only arguments can be passed by the name to such parameters when invoking a function from an imported module. These arguments can still be passed as positional arguments.

### Error Type and Constructor

- The error detail type must now belong to the `detail` type defined in the error lang library.

   ```ballerina
   public type Detail record {|
      string message?;
      error cause?;
      (anydata|error)...;
   |};
   ```

- The error constructor now accepts `detail` fields as individual named arguments as opposed to accepting a single mapping as the `detail` argument.	 

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

   Previous Syntax

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

- Concatenation with the + operator is no longer allowed between `string` values and values of other basic types. The `.toString()` method can be used on any variable to retrieve the `string` representation prior to concatenating. Alternatively, the string template expression can also be used.

   Previously Syntax

   ```ballerina
   int i = 1;
   string s = "Value: " + i;
   ```

   Alternative I

   ```ballerina
   int i = 1;
   string s = "Value: " + i.toString();
   ```

   Alternative II

   ```ballerina
   int i = 1;
   string s = string `Value: ${i}`;
   ```

# JBallerina 1.0 - 2019R3 Specification Deviations

### Lexical structure
- `QuotedIdentifier` supports only alphanumeric characters and "." [#18720](https://github.com/ballerina-platform/ballerina-lang/issues/18720).

### Values, Types and Variables.
- Limited Singleton type support [#13410](https://github.com/ballerina-platform/ballerina-lang/issues/13410).
- Limited constant support [#13944](https://github.com/ballerina-platform/ballerina-lang/issues/13944).
- `0.` syntax is not supported for floating point literals [#13168](https://github.com/ballerina-platform/ballerina-lang/issues/13168).
- Cannot use a byte in an int context even though byte is a subtype of int [#14366](https://github.com/ballerina-platform/ballerina-lang/issues/14366).
- Hex floating point literals are allowed for Decimal values [#14775](https://github.com/ballerina-platform/ballerina-lang/issues/14775).
- `StringNumericEscape` is not supported [#13180](https://github.com/ballerina-platform/ballerina-lang/issues/13180).
- Direct table type descriptor is not implemented [#13170](https://github.com/ballerina-platform/ballerina-lang/issues/13170).
- Defaultable and rest parameters are not supported with function pointers [#10639](https://github.com/ballerina-platform/ballerina-lang/issues/10639).
- Filler values for finite types are not supported [#13612](https://github.com/ballerina-platform/ballerina-lang/issues/13612).
- Object initialization does not follow the initialization protocol outlined in the spec [#15240](https://github.com/ballerina-platform/ballerina-lang/issues/15240).
- Object __init() allows using self other than to access/modify a field when there are potentially uninitialized fields [#17917](https://github.com/ballerina-platform/ballerina-lang/issues/17917).
- Default values of fields and default values of defaultable function parameters of abstract objects are not available when referencing the objects in other objects [#18405](https://github.com/ballerina-platform/ballerina-lang/issues/18405).
- Constraints are mandatory for future, stream and typedesc [#17922](https://github.com/ballerina-platform/ballerina-lang/issues/17922).
- Inferred error type (`error<*>`) is not allowed [#18007](https://github.com/ballerina-platform/ballerina-lang/issues/18007).
- Type narrowing does not happen with equality checks [#18167](https://github.com/ballerina-platform/ballerina-lang/issues/18167).

### Expressions
- Reference equality checks produce incorrect results for float NaN/NaN and -0.0/+0.0 [#11913](https://github.com/ballerina-platform/ballerina-lang/issues/11913).
- Constants cannot be used to specify the length of a fixed length array [#13162](https://github.com/ballerina-platform/ballerina-lang/issues/13162).
- Incorrect type inferring for int-literals/floating-point-literals used in equality expressions [#13904](https://github.com/ballerina-platform/ballerina-lang/issues/13904).
- Numeric conversion does not happen when casting to a finite type [#14373](https://github.com/ballerina-platform/ballerina-lang/issues/14373).
- The `typedesc` returned by `typeof` is not in line with the spec [#15278](https://github.com/ballerina-platform/ballerina-lang/issues/15278).
- Compilation fails for valid a mapping constructor when there is no contextually expected type [#17186](https://github.com/ballerina-platform/ballerina-lang/issues/17186).
- Optional field access is not allowed on a union of record and map [#17942](https://github.com/ballerina-platform/ballerina-lang/issues/17942).
- The rest arg is restricted to the rest parameter [#17943](https://github.com/ballerina-platform/ballerina-lang/issues/17943).
- Integer division does not panic on overflow [#17969](https://github.com/ballerina-platform/ballerina-lang/issues/17969).
- Certain floating point numerical comparisons with `NaN` return incorrect results [#17977](https://github.com/ballerina-platform/ballerina-lang/issues/17977).
- Reference equality checks produce incorrect results for equal decimal values with different precision [#17984](https://github.com/ballerina-platform/ballerina-lang/issues/17984).
- Interpolation should not be allowed in the value of a namespace attribute [#18938](https://github.com/ballerina-platform/ballerina-lang/issues/18938).

### Actions and Statements
- Local type definition statements are not supported [#17946](https://github.com/ballerina-platform/ballerina-lang/issues/17946).
- Multiple receive action is not supported [#18163](https://github.com/ballerina-platform/ballerina-lang/issues/18163).
- Final local variable declarations without an init expression are not supported [#15044](https://github.com/ballerina-platform/ballerina-lang/issues/15044).
- Match pattern lists with multiple match patterns (OR) are not supported for structured patterns [#13949](https://github.com/ballerina-platform/ballerina-lang/issues/13949).
- Partial support for list, mapping and error match patterns [#15962](https://github.com/ballerina-platform/ballerina-lang/issues/15962).
- Compatible map to record assignment is not allowed [#17202](https://github.com/ballerina-platform/ballerina-lang/issues/17202).
- List binding patterns don’t work with arrays [#17927](https://github.com/ballerina-platform/ballerina-lang/issues/17927).
- Not all actions are allowed to be nested inside other actions [#17993](https://github.com/ballerina-platform/ballerina-lang/issues/17993).
- Inferred type for final variables declared with `var` is not the precise type [#18166](https://github.com/ballerina-platform/ballerina-lang/issues/18166).
- Asynchronous send is not an action (currently a statement) [#18639](https://github.com/ballerina-platform/ballerina-lang/issues/18639).

### Module-Level Declarations
- Versioned imports are not supported [#4087](https://github.com/ballerina-platform/ballerina-lang/issues/4087).
- Listener declarations without the type-descriptors are not supported [#18200](https://github.com/ballerina-platform/ballerina-lang/issues/18200).

### Module and Program Execution
- A program exits even when there are listeners started (currently looks for services) [#18601](https://github.com/ballerina-platform/ballerina-lang/issues/18601).

### Metadata
- Annotation types (mapping types) are not restricted to `anydata` and thus, non-constant annotation values are not constructed as readonly values [#15533](https://github.com/ballerina-platform/ballerina-lang/issues/15533).
- Annotation attachment is not supported for anonymous function expressions, local variable declaration statements and local type definition statements [#18207](https://github.com/ballerina-platform/ballerina-lang/issues/18207).
- An annotation is not available to to indicate that a newly created strand should be in a separate thread from the current strand [#18001](https://github.com/ballerina-platform/ballerina-lang/issues/18001).

### Lang Library
- Creating an immutable clone of a container does not narrow its inherent type to a type that consists of just its current shape [#13189](https://github.com/ballerina-platform/ballerina-lang/issues/13189).
- Use of stack-like methods and queue-like methods on fixed-length arrays/tuples is not  checked at compile time [#18662](https://github.com/ballerina-platform/ballerina-lang/issues/18662).
- The lang.table module contains functions not defined in the specification [#18869](https://github.com/ballerina-platform/ballerina-lang/issues/18869).
- The lang.xml module contains functions not defined in the specification [#18870](https://github.com/ballerina-platform/ballerina-lang/issues/18870).
- The lang.map module functions which modify the value are disallowed on records [#18873](https://github.com/ballerina-platform/ballerina-lang/issues/18873).
- The lang.array module functions which modify the value are disallowed on tuples [#18874](https://github.com/ballerina-platform/ballerina-lang/issues/18874).

### Preview Features
- XML access expressions are not defined in the spec. [#18875](https://github.com/ballerina-platform/ballerina-lang/issues/18875).
