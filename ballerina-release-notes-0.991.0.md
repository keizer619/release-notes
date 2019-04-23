# Overview of Ballerina 0.991.0
Ballerina 0.991.0 consists of improvements to the language syntax and semantics based on the language specification and new features and improvements to the standard library modules, extensions, and tooling.


# Highlights

- Improved error handling with new checkpanic expression. Now, error values cannot be ignored using "_".
- Improved type checking with the union type.
- Improved syntax for the closed record, closed arrays, and string/XML interpolation etc.
- Additionally, this release includes several fixes to address Ballerina language specification deviations from the Ballerina 0.990 specification.

# Breaking Language Changes

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

- Error values can no longer be ignored using “_”.  If the result of an expression is an `error` type or is a union that contains an `error` type, the result cannot be ignored using “_”. This is to enforce the user to handle all possible errors.

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


# Breaking Standard Library Changes

## Crypto Library Changes
- Ballerina crypto stdlib was made flexible enough to introduce new features and algorithms in future.

*Previous syntax*
```ballerina
string input = "Hello Ballerina";
crypto:hash(input, crypto:MD5)
```

*New syntax*
```ballerina
string input = "Hello Ballerina";
byte[] inputArr = input.toByteArray("UTF-8");
byte[] output = crypto:hashMd5(inputArr);
```

Ballerina gRPC client initialization in the generated code is changed. Generated file(*.pb.bal) from previous versions of Ballerina distributions will not be worked with this release. Regenerate the client stub using proto to Ballerina tool and replace the previously generated file with the new one.

## Task Library Changes
- Redesigned the Task library to function as a service. The previous task implementation of `task:Timer` and `task:Appointment` will be deferred and new `task:Listener` and `task:Scheduler` objects are introduced to create task timers and appointments.

Tasks now can be defined as a service and as an object. Both the previously existed Timer and Appointment functionalities can be used by new Listener and/or Scheduler.

### Timer
Timer was created by providing `onTrigger` and `onError` functions along with the `interval` and `delay` parameters. Now `interval` and `delay` parameters are provided through a `task:TimerConfiguration` record type, while `onTrigger`  function should be implemented as a resource function inside the attaching service.

*Previous Syntax*
```ballerina
function startTimer() {
	((function) returns error?) onTriggerFunction = cleanup;
function (error) onErrorFunction = cleanupError;
timer = new task:Timer(onTriggerFunction, onErrorFunction, 1000,
delay = 500);
	timer.start();
}

function cleanup() returns error? {
	// onTriggerFunction
}

function cleanupError(error e) {
	// handle error
}
```

*New Syntax*
```ballerina
function startTimer() {
	task:TimerConfiguration timerConfiguration = {
        interval: 1000,
        initialDelay: 500
    }
    task:Scheduler timer = new(timerConfguration);
    var attachResult = timer.attach(timerService);
    if (attachResult is error) {
        // handle error
    } else {
        var startResult = timer.start();
        if (startResult is error) {
            // handle error
        } else {
            // successfully started timer
        }
    }
}

Service timerService = service {
	resource function onTrigger() {
		// onTrigger function
	}
}

```

OR

```ballerina
task:TimerConfiguration timerConfiguration = {
   interval: 1000,
   initialDelay: 3000
};
listener task:Listener timer = new(timerConfiguration);
Service timerService on timer {
	resource function onTrigger() {
		// onTrigger function
    }
}
```

### Appointment
`task:Appointment` functionality can be implemented using `task:Listener` and the `task:Scheduler` objects. Previously the `cronExpression` used for the appointment given as a parameter, along with the `onTrigger` and `onError` functions. Now the `cronExpression` should be provided using `appointmentDetails` field inside the  `task:AppointmentConfiguration` record type. Alternatively, `task:AppointmentData` record can also be provided as the `appointmentDetails` field (see below examples).

*Previous Syntax*

```ballerina
function startAppointment() {
	((function) returns error?) onTriggerFunction = cleanup;
    function (error) onErrorFunction = cleanupError;
    appointment1 = new task:Appointment(onTriggerFunction, onErrorFunction, "0/2 * * * * ?");
        appointment.schedule();

    }

function cleanup() returns error? {
	// onTrigger function
}

function cleanupError(error e) {
	// handle error
}
```

*New Syntax*
```ballerina
task:AppointmentData appointmentData = {
    seconds: "0/2",
    minutes: "*",
    hours: "*",
    daysOfMonth: "?",
    months: "*",
    daysOfWeek: "*",
    year: "*"
};

task:AppointmentConfiguration appointmentConfiguration = {
	appointmentDetails: appointmentData
};

listener task:Listener appointment = new(appointmentConfiguration);

service appointmentService on appointment {
	resource function onTrigger() {
		// onTriggerFunction
	}
}

```
OR

```ballerina
function startAppointment() {
    task:AppointmentConfiguration appointmentConfiguration = {
            appointmentDetails: “0/2 * * * * ?”
    };
    task:Scheduler appointment = new(appointmentConfiguration);
    var attachResult = appointment.attach(appointmentService);
    if (attachResult is error) {
        // handle error
    } else {
        var startResult = appointment.start();
        if (startResult is error) {
            // handle error
        } else {
            // successfully started the appointment
        }
    }
}

service appointmentService = service {
	resource function onTrigger() {
		// onTrigger function
	}
}
```


## Time Library Changes

The `format`, `parse`, `createTime`and `toTimeZone`  functions are changed to return errors when there is an issue.

*Previous Syntax*

```ballerina
time:Time timeVal = time:parse("2016-03-01T09:46:22.444", yyyy-MM-dd'T'HH:mm:ss.SSS");
```

*New Syntax*
```ballerina
time:Time|error timeVal = time:parse("2016-03-01T09:46:22.444", yyyy-MM-dd'T'HH:mm:ss.SSS");
```


# What's new in Ballerina 0.991.0

## Language & Runtime

- Introduction of the `checkpanic` unary expression
  - `checkpanic` is a unary expression that is used to handle errors. The `checkpanic` expression checks the result of the sub-expression and panics if the result is `error`.
  - The only difference between the `checkpanic` expression and the `check` expression is that, if the result of the sub-expression is of type `error`, the `checkpanic` expression results in a panic while the `check` expression returns the `error` value.


- Error values can now be destructured using binding patterns and typed binding patterns as follows,
```ballerina
// Usage of error typed binding pattern.
var error (reasonString, detailMap) = getErrorValue();

// Usage of error binding pattern.
string reasonString;
map<anydata|error> detailMap;
error (reasonString, detailMap) = getErrorValue();
```

- Value equality checks (`==`, `!=`) are now allowed with `error` values.

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
- Introduced OAuth2 password grant type and client credentials grant type along with the direct token mode for outbound authentication.
- Added support for password hashing techniques for basic auth authentication.
- Introduced filepath module with a set of utility functions to manipulate the file path in a way that is compatible with the target operating system.
- Introduced an "encoding" API to perform byte[] to string conversions using different encoding mechanisms.
- Introduced RSA and AES encryption/decryption capabilities to the crypto API.
- Introduced RSA signing operations to the crypto API.
- Improved to h2/mysql/jdbc database clients to share a single, global connection pool for interacting with the same database with same connection properties. The option to use an unshared or a locally shared custom pool is provided.
- Introduced a shared global connection pool for HTTP client connectors. Per-client pools are also possible with `http:PoolConfiguration`.
- Improved the HTTP/2 client connection pool to distribute the request load among I/O threads properly.


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
