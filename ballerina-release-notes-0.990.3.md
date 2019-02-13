# Overview of Ballerina 0.990.3

Ballerina 0.990.3 is a release iteration done based on the 0.990 language specification. It includes improvements on standard library modules, extensions, tooling, and language syntax.

# Compatibility and Support

The record `rest-fields` descriptor should now be followed by a semi colon.
type Person record {
    string name;
    string...;
};
Constrained JSON is no longer supported. The stamp() or convert() methods can be used instead depending on the requirement.
Binary integer literals are no longer supported.
Use of `var` in the left hand side, with iterable operations ending with `map()` or `filter()` operations is disallowed.
int[] numbers = [-5, -3, 2, 7, 12];

```ballerina
// The following now results in a compilation error.
var filtered = numbers.filter(function (int i) returns boolean {
    return i >= 0;
});
```

# Improvements

## Language & Runtime

In Ballerina, floating point types (`float` and `decimal`) adheres to the IEEE754-2008 standard. Hence, floating point types accommodate NaN and Infinity concepts. Now, along with the ‘float’ type, ‘decimal’ type also supports NaN and Infinity concepts.

Dividing a number by zero will no longer result in a panic situation. Rather, it will result in infinity(+/-) for all non-zero real numbers and NaN. Now, in addition to this, the `decimal` type supports three built-in functions namely `isNaN()`, `isInfinite()`, and `inFinite()` to check whether a given number is NaN, Infinity, or a finite number.

## Standard Library

- Now, the type ‘time:Time’, which represents an instance of time with the associated timezone is a ‘record type’ (not an ‘object type’). Member functions of the previous “time:Time” object are now provided as utility functions.
- Support for using the `optional` configuration for client authentication in SSL.
- The SimpleDurableTopicSubscriber, SimpleQueueReceiver, SimpleQueueSender, SimpleTopicPublisher, and SimpleTopicSubscriber were removed from the JMS API. The initialization API of the TopicPublisher, TopicSubscriber, DurableTopicSubscriber, QueueReceiver, and QueueSender has been modified to support all `Simple` use cases as well.
- Support on WebSub Hub persistence.
- Basic auth support for WebSub Hub.
- Ballerina `crypto` standard library is reorganized to enhance the extensibility of the library and to increase reusability across other standard libraries.
  - Now, the `crypto` standard library provides RSA-signing capabilities in addition to hashing operations, HMAC generation, and CRC32B checksum generation.
  - Instead of returning Hex encoded `string` values, crypto operations now return `byte[]` (byte array). This allows users to consume raw bytes as well as to use the newly added `encoding` standard library, to get string values encoded using different encoding algorithms.
  - Also, the `crypto` standard library now provides the `crypto:KeyStore`, `crypto:PrivateKey`, and  `crypto:PublicKey` records. These are usable across other standard libraries to represent key stores, private keys, and public keys.
- Ballerina `encoding` standard library provides functions to perform the following:
  - Encode `byte[]` (byte arrays) to `string` using different encoding algorithms.
  - Decode `string` values into `byte[]` (byte array) using different decoding algorithms.
  - The `byteArrayToString` function that can be used to encode byte[] into a `string` using a selected character encoding.

## IDEs & Language Server

- Update LSP version to v3.13.0.
- Markup Content Support on Signature Help, Hover Provider, and Completion.
- Code Lens support.

### IDEA Plugin

- Ballerina code folding support.
- Spell-checking support.
- Improved Signature Help.
- Minor bug fixes and improvements.

## Compiler Extensions

- Support on AWS Lambda functions.

# Performance Results

Refer Ballerina [performance test results](https://github.com/ballerina-platform/ballerina-lang/blob/v0.990.3/performance/benchmarks/summary.md) available in the repository.

# Bug Fixes

Refer [Github milestone issues](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+milestone%3A0.990.3+is%3Aclosed+label%3AType%2FBug) to view bug fixes.

# Getting Started

You can download the Ballerina distributions, try samples, and read the documentation at https://ballerina.io. You can also visit the [Quick Tour] (https://ballerina.io/learn/quick-tour/) to get started.

We encourage you to report issues, improvements, and suggestions at the [Ballerina Github Repository](https://github.com/ballerina-platform/ballerina-lang).