# Overview to Ballerina 0.990.1

Ballerina 0.990.1 release is an iteration effort that aims to stabilize the previous 0.990.0 release, which is an implementation based on the Ballerina 0.990.0 language specification.

# Improvements

## Language

Return statement is now optional for functions with optional return type signatures.

```ballerina
function foo(string txt) returns error? {
    int x = check int.convert(txt);
}
```

The above is the same as the following code.

```ballerina
function foo(string txt) returns error? {
    int x = check int.convert(txt);
    return;
}
```

Function invocation support for string literals.

```ballerina
string name = “ballerina”.toUpper();
```

There are some updates to the underlying stream processing layer of Ballerina. It introduces a few changes in the syntax.

- When accessing the stream attributes you should use the stream name as the prefix. If stream name is `x` and attribute is `a`, you should access as `x.a`.
- Support for table and stream joins.

# Bug Fixes
Please refer [Github milestone issues](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.990.1+is%3Aclosed+label%3AType%2FBug) to view bug fixes
