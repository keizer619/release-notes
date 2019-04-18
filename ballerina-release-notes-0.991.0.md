# Overview of Ballerina 0.991.0
Ballerina 0.991.0 consists of improvements to the language syntax based on the 2019r0 language specification and new features and improvements to the standard library modules, extensions, and tooling.

# Compatibility and Support
- Closed record syntax was updated to use the {| and |} delimiters to indicate that the record is closed.

*Previous syntax*
```ballerina
type Person record {
    string name;
    int age;
    !...;
};
```

*New syntax*
```ballerina
type Person record {|
    string name;
    int age;
|};
```


- The syntax for destructure binding pattern for closed records was also updated to use the {| and |} delimiters. 

*Previous syntax*
```ballerina
Person p = { name: "John Doe", age: 25 };
Person { name: pName, age: pAge, !... } = p;
```

*New syntax*
```ballerina
Person p = { name: "John Doe", age: 25 };
Person {| name: pName, age: pAge |} = p;
```

- The syntax for closed arrays with inferred length was changed as follows.

*Previous syntax*
```ballerina
string[!...] array = ["a", "b", "c"];
```
*New syntax*
```ballerina
string[*] array = ["a", "b", "c"];
```
		
- It is now mandatory for list element types to have an implicit initial value. Where a type `T` does not have an implicit initial value, a `T?[]` could be used.

 
- Use of the `null` literal is now restricted to JSON contexts.
```ballerina
json response = null; // Allowed
string? name = null; // Compile error
```

- Error values can now be destructured using binding patterns and typed binding patterns as follows,
```ballerina
// Usage of error typed binding pattern.
var error (reasonString, detailMap) = getErrorValue();

// Usage of error binding pattern.
string reasonString;
map<anydata|error> detailMap;
error (reasonString, detailMap) = getErrorValue();
```

- Introduction of the `checkpanic` unary expression
  - `checkpanic` is a unary expression that is used to handle errors. The `checkpanic` expression checks the result of the sub-expression and panics if the result is `error`.
  - The only difference between the `checkpanic` expression and the `check` expression is that, if the result of the sub-expression is of type `error`, the `checkpanic` expression results in a panic while the `check` expression returns the `error` value.

- String template expression interpolation syntax was updated.

*Previous syntax*
```ballerina
string s = string `Hello {{name}}!`;
```
*New syntax*
```ballerina
string s = string `Hello ${name}!`;
```

- XML literal interpolation syntax was updated.

*Previous syntax*
```ballerina
xml x = xml `<Person name={{name}}>{{content}}</Person>`;
```
*New syntax*
```ballerina
xml x = xml `<Person name=${name}>${content}</Person>`;
```

- Interpolating XML elements and attribute names are no longer supported.

- The XML attribute expression now directly returns the attributes of a singleton `xml` element as a  `map<string>` or `()` if the operand is not a singleton `xml`. 

*Previous syntax*
```ballerina
map<anydata> attributeMap = map<anydata>.convert(x1@);
```
*New syntax*
```ballerina
map<string>? attributeMap = x1@;
```

- Error values can no longer be ignored using “_”.

If the result of an expression is an `error` type or is a union that contains an `error` type, the result cannot be ignored using “_”. This is to enforce the user to handle all possible errors.

- The capability to consider (run) any public function as an entry point to a Ballerina program was removed. 

- The types of the parameters and the return type of the `main` function were restricted to subtypes of `anydata` and `error?` respectively.

- The diamond operator was updated to perform type-casting as follows:
```ballerina
Foo f = <Foo> value;
```
If `value` belongs to `Foo`, casting will be successful and `value` will be assigned to `f`. Else, casting will result in a panic.

- The type of the `error` detail is now restricted to be a mapping that could only have fields of types that are subtypes of `anydata|error`.

- The default record rest field type was changed to `anydata|error` from `anydata`.

- `anydata` was updated to include structures of pure values (values whose types are subtypes of `anydata|error`). `anydata` is now the union of the following: 

```ballerina
() | boolean | int | float | decimal | string | (anydata|error)[] | map<anydata|error> | xml | table

```

- Module variable declarations are not allowed to be `public` now and also require an assignment.

- The syntax for functions with external implementations has been changed as follows:

*Previous syntax*
```ballerina
extern function externFunction();
```
*New syntax*
```ballerina
function externFunction() = external;
```

- Value equality checks (`==`, `!=`) are now allowed with `error` values.

- The signature of the `__attach()` method of `AbstractListener` has been changed as follows:

*Previous signature*
```ballerina
public function __attach(service s, map<any> annotationData) returns error?
```
*New signature*
```ballerina
public function __attach(service s, string? name = ()) returns error?;
```
- Type narrowing does not apply to global variables now.


# Improvements
## Language & Runtime
- The return signature of the initializer of an object can now be `error?` (sub types of `error` allowed) or `()`.
```ballerina
type Person object {
    string name;

    function __init(string name) returns error? {
        self.name = check fnWithPotentialErrorReturn(name);
    };
}

// Usage
Person|error person = new("John Doe");
```

- Tuple type is now iterable via the `foreach` statement.
- Patterns in the static value match statement now support const values.
- String can concatenate into XML values and the string part is automatically converted to an XML character sequence, resulting in XML concatenation. This supports both '+' and '+=' operators.
```ballerina
xml xValue = xml `<name>John</name>`;
xml yValue = xValue + "suf";
yValue += "fix";
io:println(yValue);
```

## Standard Library
- Introduced NATS connector.
- Introduced Artemis connector.
- Introduced RabbitMQ connector.
- Improvement to h2/mysql/jdbc database clients to share a single, global connection pool for interacting with the same database with same connection properties. The option to use an unshared or a locally shared custom pool is provided.
- Introduced OAuth2 password grant type and client credentials grant type along with the direct token mode for outbound authentication.
- Added support for password hashing techniques for basic auth authentication.
- Introduced filepath module with a set of utility functions to manipulate the file path in a way that is compatible with the target operating system.
- Introduced a shared global connection pool for HTTP client connectors. Per-client pools are also possible with `http:PoolConfiguration`.
- Improved the HTTP/2 client connection pool to distribute the request load among I/O threads properly.
- Refactored the crypto API to introduce new features and algorithms in the future.
- Introduced an "encoding" API to perform byte[] to string conversions using different encoding mechanisms.
- Introduced RSA and AES encryption/decryption capabilities to the crypto API.
- Introduced signing operations to the crypto API.
- Redesigned the Task library to function as a service. The previous task implementation of `task:Timer` and `task:Appointment` will be deferred and new `task:Listener` and `task:Scheduler` objects are introduced to create task timers and appointments.

## IDEs & Language Server
- The Language Server Extension API to contribute custom completions.
- Improvements to the Go-To definition, Find all references and Rename features.
- Diagram view function collapsing capability. This feature allows you to expand functions in a sequence diagram to view its internal logic. 
### IDEA Plugin
- Improved performance, stability, and reduced plugin size with “Lsp4IntelliJ” language client support.
- Ballerina language server-based code formatting.
- Minor bug fixes and improvements.

## Compiler Extensions
- Support for generating OpenShift artifacts.
# Performance Results
See Ballerina [performance test results](https://github.com/ballerina-platform/ballerina-lang/blob/v0.991.0/performance/benchmarks/summary.md), which is available in the repository.

# Bug Fixes
See [Github milestone issues](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.991.0+is%3Aclosed) to view bug fixes.


# Getting Started
You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit [Quick Tour] (https://ballerina.io/learn/quick-tour/) to get started. 

We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).


