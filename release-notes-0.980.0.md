# Overview to Ballerina 0.980.0
Ballerina 0.980.0 is an iteration for Ballerina 0.970.0, which was released previously. This release focused on stabilizing the platform. Additionally, there are type system improvements done on the language.

# Compatibility and Support

## Closed and Open Records

Now records may contain extra fields, that is fields other than those named by individual type descriptors in the record type definition. By defaults, records can contain extra fields with `any` value.

```ballerina
type Person record {
    string name,
    int age = 10,
};

...
// The "country" is an extra filed, which is not defined in the person type descriptor.
Person tom = { name : "tom", age : 20, country : "USA"};

// You can access "country" field similar to other fields, but return type will be any.
any country = tom.country; // or use tom["country"]
```

Extra fields can be defined by using an optional `record rest descriptor`; `RecordRestType...` at end of the record definition. In the above example, the Person record definition is equivalent to the definition with `any...`.

```ballerina
type Person record {
    string name,
    int age = 10,
    any...
};
```
In the following record definition, the “rest fields” are constrained to `string`.

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

If RecordRestType is `!`, then the record may not contain any extra fields. 

```ballerina
type Person record {
    string name,
    int age = 10,
    !... // indicates that additional fields are not allowed
};
...

Person tom = { name : "tom", age : 20, country : "USA"}; // This is a compile time error.
```

A record type including !... is called `closed`; a record type that is not closed is called `open`.


## Fixed length Arrays

Now the length of an array can be fixed by providing the array length with the array type descriptor. 

```ballerina
int[5] array1 = [2, 15, 200, 1500, 5000];
int[5] array2; // Creates an integer array of size five, filled with default integer values
```
An array length of `!...` means that the length of the array is to be implied from the context; as shown below:

```ballerina
int[!...] sealedArray = [1, 3, 5]; // Creates a sealed integer array of size 3.
```

## Object Syntax change.

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

## Byte type and Blob type Change

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

## Table literal change

## Map access change
Values of a map can be accessed using index-based syntax as well as field-access syntax. These two syntaxes now behave differently. Getting a value using field-access syntax returns the value if the key exists. Otherwise a runtime error is thrown. Index-based syntax also will return the value if the key exists. However, it will return a null value if the key does not exist.
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
## Runtime
- improvement 1 

## Standard Library
-  **WebSocket**
	- The signatures of the `onBinary`, `onPing`, and `onPong` resources were changed due to removal of the `blob` type. These resources now have `byte[]` in their signature instead of a blob.
	- The method signatures of  `pushBinary()`,  `ping()`, and `pong()` have also been changed to take the `byte[]` input parameter instead of a blob.

- The HTTP transport error handler has been improved so that it recovers execution from inbound/outbound failures such as idle socket timeout and abrupt connection closure. 

-  **Circuit Breaker**
     - Introduce requestVolumeThreshold parameter support
This parameter sets the minimum number of requests in a `RollingWindow` that will trip the circuit. So the rollingWindow configurations can be specified as follows.

``` ballerina
rollingWindow: {
            timeWindowMillis: 10000,
            bucketSizeMillis: 2000,
            requestVolumeThreshold: 10
        }
```

## Build & Package Management
### CLI
- Enhance build output
- Integrate test execution to build
- Mandate build on push
### Central 
- View previous versions of a package.
- Show ballerina compatibility section.
## IDEs & Language Server
	### Composer
	- Shipping composer as a native Electron App.
	### Language Server
	- Introducing source code formatting.
- Supporting find all symbols in a document and in the workspace.
	### IntelliJ IDEA
- Improve debugger
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

# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour] (https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).