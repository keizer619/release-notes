# Overview to Ballerina 0.983.0

This release consist of significant changes on the Ballerina distribution. The name of the distribution has been changed from Ballerina Platform to Ballerina. The Composer is not shipped with the distribution anymore and its functionalities are now included in the VSCode plugin directly.

# Compatibility and Support

Ballerina distribution now consists of Ballerina language and toolings components. This change is also reflected in the Docker hub images. From this release onwards, the [Ballerina Docker](https://hub.docker.com/r/ballerina/ballerina) image consists of Ballerina language and tooling components. Ballerina runtime is available in an image named [ballerina-runtime](https://hub.docker.com/r/ballerina/ballerina-runtime).

## Syntax Changes

**Object type reference syntax**
Object type references provide a way to copy the members from an abstract object into another object. It is equivalent to specifying the members explicitly within the new object.

```ballerina
type Person abstract object {
    public int age;
    public string firstName;
    public string lastName;

    function getFullName() returns string;
};

type Manager object {
    *Person;    // Referencing 'Person' object.
    *Owner;    // Referencing another object.

    public string dpt;

    function getFullName() returns string {
        return firstName + "  " + lastName;
    }
};
```

**Optional fields in records**

Fields in records can now be marked as optional by adding `?` to the end of the identifier. When creating a record, optional fields can be omitted. Omitting a required field results in a compile error. Fields with implicit initial values or explicitly defined default values are effectively optional fields even if they are not marked as optional. Explicit default values cannot be set to optional fields.

```ballerina
type Gender "male"|"female";

type Person record {
   string fname = "default"; // required field with explicit default value
   string lname; // required field with implicit initial value
   Gender gender; // non-defaultable required field
   function (Person) returns string details?; // optional field with no implicit initial value
};

public function main() {
   Person p = {gender: "male"};
   p.details = function (Person p) returns string {
       return "[" + p.fname + " " + p.lname + "]";
   };
}
```

**Table syntax**
The keyword used for the primary key of a table has changed to “key”.

```ballerina
table<Employee> tbEmployee = table {
        { key id, name, salary },
        [ { 1, "Mary",  300.5 },
        { 2, "John",  200.5 },
        { 3, "Jim", 330.5 }
        ]
};
```

**Post increment and decrement operators**
These were statements in Ballerina and there was no siginificant improvement as these cannot be used as expressions. Therefore, these statements have been removed to simplify usage.

Old syntax

```ballerina
int i = 1;
i++;
```

Alternative syntax

```ballerina
int i = 1;
i += 1;
```

# Improvements

## Standard Library

- Support JSON array as a valid data binding type
- Support little-endian network byte order (data-io)
- JDBC module has been moved from the Ballerina organization to the Ballerinax organization. 
              When importing the module, users now need to import it as `ballerinax/jdbc`
- Support the contract first approach in gRPC service creation.
- Support byte[] field type in gRPC.
- The HTTP/2 error handling has been improved so that it recovers execution from inbound/outbound failures.
- Add support to integrating LDAP user stores 
- Introduce retrying over HTTP status codes for HTTP Client
- Backpressure handling for HTTP 1.x
- Introduce functions to retrieve topic/subscriber details from a WebSub Hub
- `http:SecureListener` and `http:APIListener` were removed. Relevant functionalities were consolidated into `http:Listener`.
- `propagateToken` field of `http:AuthProvider` was renamed to `propagateJwt`.
- During JWT signature validation, validity of the public key certificate is also checked.

## IDE Tooling

From this release onwards, Composer is removed from the distribution. You can view and edit Ballerina programs visually using the VSCode plugin.

### VSCode

- Run/Debug Ballerina tests
- Doc preview - Shows generated API Doc of a source file in real time
- Script arguments and command options for the `launch.json`.

```ballerina
    "scriptArguments": ["foo", "bar"],
    "commandOptions": ["-e", "b7a.log.level=DEBUG"]
```

### IDEA

- In-built language server client
- Improved hover support

### Language Support

The following new features are now supported in both VSCode and IDEA plugins.
- `Pull Module`, code action support for the unfetched module imports.
- Completion support for a module’s functions, Records, Objects, etc. even without the module import.
- Auto import modules on completion of item selection.
- Updated icons for completion of item categories.
- Improved source formatting.

## Extensions

### Kubernetes

- Allow creating Resource Quotas.
- Enable copying directories.
- Allow creating deployments using `main` functions.
- Support reference values for environment variables in deployments.
- Allow setting the `sessionAffinity` field for services.

# Performance Results

Please refer to [Ballerina performace test results](https://github.com/ballerina-platform/ballerina-lang/blob/v0.983.0/performance/benchmarks/summary.md) that is available in the repository.

# Bug Fixes

Please refer [Github milestone issues](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.983.0+is%3Aclosed+label%3AType%2FBug) to view bug fixes.

# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour](https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).
