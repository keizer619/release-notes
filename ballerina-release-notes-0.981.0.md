# Overview to Ballerina 0.981.0

This release has considered about further stabilizing previous release with bug fixing effort. Apart from that there is a major improvement on Websocket standard library with the introduction of supporting union type for `pushText()` and multiple type support for `onText()` functions. Also there are few language syntax changes.

# Compatibility and Support

The `native` keyword has been changed to `extern` due to Ballerina moving towards a model of native compilation. With this model, code will be an external library and everything is native. Libraries can be written or not written in Ballerina.

Old syntax:

``` ballerina
public native function string::toUpper() returns string;
```

New syntax:

``` ballerina
public extern function string::toUpper() returns string;
```

# Improvements

## Language & Runtime

- Bitwise complement (~) operator inverts the bits of its operand expression. The static type of the operand must be int, and the static type of the result is an int.

```ballerina
int i = 387465;
int j = ~i;
```

- Introduce index based access for tuples. Elements of a tuple type can now be accessed via indexes as follows:

```ballerina
(boolean, int, string) x = (true, 3, "abc");
boolean tempBool = x[0];
string tempString = x[2];
x[1] = 4;
```

## Standard Library

- **WebSocket**

- The `pushText` function takes a union type instead of a string type. The new signature is as follows: 

``` ballerina
public function pushText(string|json|xml|boolean|int|float|byte|byte[] data, boolean final = true) returns error? 
```

- The `onText` function can take a string, JSON, XML, record, or byte[] types instead of only string. The `onText` signature can be any one of the following:

``` ballerina
onText(endpoint caller, string text, boolean final)
onText(endpoint caller, json jsonVal)
onText(endpoint caller, xml xmlVal)
onText(endpoint caller, Person person)
onText(endpoint caller, byte[] data)
```

> **Note**: In the above code snippet, Person is a record type.

For all values except for string type, the values are aggregated if a continuation frame is received. Hence the value of final is not relevant in the case of other types except string.

## IDEs & Language Server

### Composer

- Sealed types source generation support.

### Language Server

- Nil lifting completion support.

# Bug Fixes

Please see [Github milestone](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.981.0+is%3Aclosed+label%3AType%2FBug) for bug fixes.

# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour] (https://ballerina.io/learn/quick-tour/) to get started. We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).
