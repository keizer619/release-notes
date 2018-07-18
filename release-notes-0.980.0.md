# Overview to Ballerina 0.980.0
Ballerina 0.980.0 is an iteration for Ballerina 0.970.0, which was released previously. This release focused on stabilizing the platform. Additionally, there are type system improvements done on the language.

# Compatibility and Support

## Closed and Open Records

Now records may contain extra fields, that is, fields other than those named by individual type descriptors in the record type definition. By default, records can contain extra fields with `any` value.

```ballerina
type Person record {
    string name,
    int age = 10,
};

...
// The "country" is an extra field, which is not defined in the person type descriptor.
Person tom = { name : "tom", age : 20, country : "USA"};

// You can access the "country" field similar to other fields, but the return type will be `any`.
any country = tom.country; // or use tom["country"]
```

Extra fields can be defined by using an optional `record rest descriptor`; `RecordRestType...` at end of the record definition. In the above example, the **Person record** definition is equivalent to the definition with `any...`.

```ballerina
type Person record {
    string name,
    int age = 10,
    any...
};
```
In the following record definition, the **rest fields** are constrained to `string`.

```ballerina
type Person record {
    string name,
    int age = 10,
    string… // the rest descriptor
};

...
Person tom = { name : "tom", age : 20, country : "USA"};
string country = tom.country;
```

If `RecordRestType` is `!`, then the record may not contain any extra fields. 

```ballerina
type Person record {
    string name,
    int age = 10,
    !... // indicates that additional fields are not allowed
};
...

Person tom = { name : "tom", age : 20, country : "USA"}; // This is a compile time error.
```

A record type including `!...` is called `closed`; a record type that is not closed is called `open`.


## Fixed Length Arrays

Now the length of an array can be fixed by providing the array length with the array type descriptor. 

```ballerina
int[5] array1 = [2, 15, 200, 1500, 5000];
int[5] array2; // Creates an integer array of size five, filled with default integer values
```

An array length of `!...` means that the length of the array is to be implied from the context as shown below:

```ballerina
int[!...] sealedArray = [1, 3, 5]; // Creates a sealed integer array of size 3.
```

## Object Syntax Change.

Object type descriptor now has some syntax changes. 
1 - Object field definition change. 

```ballerina
public type Person object {
      public string name;  //visible everywhere
      private int age;        //visible only within object and its member functions
      string email;            //visible only within same package
};
```

2 - Member functions defined outside of the object should not specify any visibility modifiers
3 - New addition of “private” visibility modifier to object fields and member functions

## Byte Type and Blob Type Change

The `byte` type represents the set of 8-bit unsigned integers. The implicit initial value of the `byte` type is `0`. Value space for `byte` is 0-255 both inclusive. 

The following is an example of byte definition.

```ballerina
byte c = 23;
```

Along with general byte array type, there is also a special syntax for defining base64 and base16 based array of bytes. With this, blob type is replaced by byte array.

```ballerina
byte[] arr1 = [5, 24, 56, 243];
byte[] arr2 = base16 `aeeecdefabcd12345567888822`;
byte[] arr3 = base64 `aGVsbG8gYmFsbGVyaW5hICEhIQ==`;
```

## Bitwise AND (&), OR (|), XOR (^)
Above bitwise operations have been added for `byte` type and `int` type with the following rules.

Both the right-hand-side and left-hand-side of the expression should be of the same type (`byte` or `int`), and the expected type will be also of the same type. If this is not the case, it will result in a compilation error.

An explicit conversion operation should be applied if the type of one side is not the same as the other side.

```ballerina
byte a = 13;
byte b = 45;
byte c = a & b;
byte d = a | b;
byte e = a ^ b;
```

## Table Expression Change

The table expression has changed to support the following syntax. A table is intended to be similar to the table of a relational database table. A table value contains an immutable set of column names and a set of data rows.

```ballerina
table<Person> t1 = table {
	{ primarykey id, primarykey salary, name, age, married }, [
		 {1, 300.5, "jane",  30, true},
		 {2, 302.5, "anne",  23, false},
		 {3, 320.5, "john",  33, true}
	]
};
```

```ballerina
Person p1 = { id: 1, age: 30, salary: 300.50, name: "jane", married: true };
Person p2 = { id: 1, age: 30, salary: 300.50, name: "jane", married: true };

table<Person> t1 = table {
	{ primarykey id, salary, name, age, married },
	[p1, p2]
};
```

## Map Access Change
Values of a map can be accessed using index-based syntax as well as field-access syntax. These two syntaxes now behave differently. Getting a value using field-access syntax returns the value if the key exists. Alternately, a runtime error is thrown. Index-based syntax also will return the value if the key exists. However, it will return a null value if the key does not exist.

This would also mean that, for a constrained map, the type of the return value for the index-based syntax is always the `constraint_type|()`.

``` ballerina
map<string> m = {"fname" : "John", "lname" : "Doe"}

// Field based access
string firstName = m.fname;
string middleName = m.mname;     // runtime error

// Index based access
string? firstName = m["fname"];
string? middleName = m["mname"];     // returns null
```

# Improvements

## Standard Library
-  **WebSocket**
	- The signatures of the `onBinary`, `onPing`, and `onPong` resources were changed due to removal of the `blob` type. These resources now have `byte[]` in their signature instead of a blob.
	- The method signatures of  `pushBinary()`,  `ping()`, and `pong()` have also been changed to take the `byte[]` input parameter instead of a blob.

- The HTTP transport error handler has been improved so that it recovers execution from inbound/outbound failures such as idle socket timeout and abrupt connection closure. 

-  **Circuit Breaker**
     - Introduced `requestVolumeThreshold` parameter support. This parameter sets the minimum number of requests in a `RollingWindow` that will trip the circuit. So the rollingWindow configurations can be specified as follows.

``` ballerina
rollingWindow: {
            timeWindowMillis: 10000,
            bucketSizeMillis: 2000,
            requestVolumeThreshold: 10
        }
```

## Build & Package Management
### CLI
- Enhance build output.
- Integrate test execution to build.
- Mandate build on push.
### Central 
- View previous versions of a package.
- Show Ballerina compatibility section.
## IDEs & Language Server
### Composer
- The Composer is now shipped as a native Electron App.
### Language Server
- Source code formatting is introduced.
- The ability to find all symbols in a document and in the workspace is now supported.
### IntelliJ IDEA
- Improvements have been made to the debugger
## Ballerina Observability
- Introduced APIs such that developers can define their own trace blocks and metrics.
- Developers can attach the trace information of their code block to the default Ballerina traces, or a new trace.

```ballerina
   //Create and attach span to the default Ballerina request trace.
   int spanId = check observe:startSpan("Child Span"); 
       // Do Something
   _ = observe:finishSpan(spanId);
  
   //Create a completely new trace.    
   int spanId = observe:startRootSpan("Parent Span");
       //Do Something
   int spanId2 = check observe:startSpan("Child Span", parentSpanId = spanId);
       // Do Something
   _ = observe:finishSpan(spanId2);
       // Do Something
   _ = observe:finishSpan(spanId);
```

- Developers can create a metric (counter or gauge) and have have their own measurements. A counter is a cumulative metric that represents a single monotonically increasing value. Gauge is a single numerical value that can arbitrarily go up and down, and also based on the statistics configurations provided to the gauge, it can also report the statistics such as max, min, mean, percentiles, etc. The created metric can be registered in order to include its measurements to reporters such as Prometheus.

```ballerina
   //Create counter and register.
   map<string> counterTags = { "method": "GET" };
   observe:Counter counterWithTags = new ("CounterWithTags", desc = "Some description", tags = counterTags);
   counterWithTags.register() but {
       error e => log:printError("Cannot register the counter", err = e)
   };
  
   //Create gauge and register.
   //Create statistics config to enable statistics calculation.
   observe:StatisticConfig[] statsConfigs = [];
   observe:StatisticConfig config = {timeWindow:30000, percentiles:[0.33, 0.5, 0.9, 0.99], buckets:3};
   statsConfigs[0]=config;
          
   observe:Gauge gaugeWithStats = new ("GaugeWithTags", desc = "Some description",
                                           tags = gaugeTags, statisticConfig = statsConfigs);                                       
```

- All metrics registered can be retrieved and looked up individually.

```ballerina
   //Get All Metrics
    observe:Metric[] metrics = observe:getAllMetrics();
    foreach metric in metrics {
       //do something.
    }
   
    //Look up a registered metric.
     map<string> tags = { "method": "GET" };
        observe:Counter|observe:Gauge|() metric = observe:lookupMetric("MetricName", tags = tags);
        match metric {
            observe:Counter counter => {
                    counter.increment(amount=10);
            }
            observe:Gauge gauge => {
                    gauge.increment(amount = 10.0);
            }
            () => {
                   io:println("No Metric Found!");
            }
        }
```


# Bug Fixes

Please refer [Github milestone](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.980.0+is%3Aclosed+label%3AType%2FBug) to view bug fixes



# Specification Deviations in 0.980.0 Release.

## Values,Type and Variable

### Nil
Use of "null" value in a non-JSON related context is not restricted yet. 

### Int
Binary and Octal literal support is available in Ballerina 0.980.0 and it is not part of the specification.

### Decimal
Type `decimal` is not supported yet.

### String
- Symbolic string literal is not supported yet. 
- String iteration is not supported yet.

### Record
- Record iteration is not supported yet. 
- Required and Optional fields syntax in a record are not supported yet. 
- Record Type reference (*T) in a record type descriptor is not supported yet.
- Record type descriptor allows default values for fields, but the speciation does not allow default values.

### Table
Table type descriptor is not supported yet. 

### Error
Error type is not supported yet. Instead the implementation uses a record based error type.

### XML
In an XML sequence, a character set is treated as a single string. However, in the specification, each character item is represented by a string with a single code point.

### Object
- Abstract object type support exists, but there are some syntax changes in the new specification. 
- Object field descriptor allows default values, but the specification does not indicate that default values are allowed.
- The order of fields, methods, and constructors in object types is no longer constrained according to the specification.
- Object fields and method names are in the same namespaces, whereas the specification mentions that they are in separate namespaces. 
- In the runtime, the object's fields and methods are implicitly in-scope. However, the specification indicates otherwise. Hence, in runtime, the "self" keyword is not required to access the object’s fields within an object body.
- External member function definition uses `::` instead of `.` currently.
- Object type reference is not supported yet. 

### Singleton Types
- Runtime allows float values as a singleton type.

### Union types
- The implicit initial value generation for union types is also not complete

### Built-in object types
- Iterator type is not supported yet. 
- Iterable and collection interfaces are not implemented yet.


## Expressions
- Table constructor expression.
   - For column constraint, `primarykey` is used instead of `key`. Other constraints are not supported.
- Error constructor expression is not supported.
- XML attributes expression
   - The result type is not `map<string>`. It needs to be explicitly cast to `map` type to use.
- Built-in methods are not supported/implemented yet.
- Anonymous function syntax has changed in the specification. Instead the specification provides two alternatives syntaxes: Anonymous function expression and arrow function expression. Both are not supported.
- Unary ~ operator is not supported.
- Additive expression for `string` and `xml` is not supported.
- Shift expression for signed right shift is not supported. 
- Table query expressions are not supported.


## Statements

### Variable Definition
- A variable defined without initializing must be checked and verified that it has been assigned at each point that the variable is referenced. This validation is not yet supported.

### Compound Assignment Statement
- The compound operators  &=,  |=, ^=,  <<=,  >>=, >>>= are not yet supported.

### Destructuring Assignment Statement
- Only tuple-binding-patterns (i.e., (p1, p2, ..., pn) ) are allowed in the lhs of a destructuring assignment statement. Other types of patterns (record-binding-patterns and error-binding-patterns) are not yet supported.

### Checked Statement
- Check construct is supported for both statements as well as expressions in the current release. However, the specification indicates that it is only allowed to be used in the statement format.
- Check construct currently throws an error if the associated expression returns an error. According to specification, if the associated expression returns an error, check constructs should return that error value as a result of the containing function, similar to a return statement.

### Match Statement
- Use of `var` in a type-binding-pattern of a match-clause is not yet supported.
- Only simple-binding-patterns are supported in a type-binding-pattern of a match-clause.

### Foreach Statement
- Type descriptor is not allowed in the the binding-pattern of the foreach statement.

### Fork-Join
- Enclosing parentheses are required for the join-condition in the current fork-join syntax, where it is not required according to the specification.

### Forever statement 
- For the time-scale in the forever statement, ‘minute’, ‘hour’, ‘day’, ‘month’, and ‘year’ is supported in the current release. This is not stated in the specification.

### Transaction Statement
- Current transaction statement includes a ‘with’ keyword, but this has been removed from the specification.


## Other Deviations

- The specification suggests that there are no longer any implicit numeric conversions. However, the runtime supports `int` to `float` implicit numeric conversions.
- Documentation String and Ballerina Flavored Markdown are not supported yet. The runtime supports only documentation node `documentation { }` syntax only.
- Deprecated construct has been removed in the specification.
- Forward recovery support is not added yet. 
- The ‘native’ keyword is used to identify a function whose implementation is not provided in the Ballerina source module.
- In a function call or method call, named arguments have changed to use `:`. However, runtime uses `=`.
- The `lengthof` unary expression has been removed in the specification. However, runtime supports it.
- A function or method can be defined as `extern`. The `native` keyword has been removed. However, runtime uses the `native` keyword.


# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour] (https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).
