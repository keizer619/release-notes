# Overview of jBallerina 1.1.2

Ballerina 1.1.2 is a patch release iteration of its previous 1.1.1 release. Please refer to the Github repository for
 a list of [bug fixes](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+label%3AType%2FBug+milestone%3A%22Ballerina+1.1.2%22+is%3Aclosed+) 
 and [improvements](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A%22Ballerina+1.1.2%22+is%3Aclosed+label%3AType%2FImprovement+).

You can use the inbuilt updating capability (introduced in jBallerina 1.1.0) to update to jBallerina 1.1.2 as follows.

### For existing users:

If you are already using **jBallerina version 1.1.0 or 1.1.1**, you can directly update your distribution to jBallerina
 1.1.2 by executing the below command:
 
> `ballerina dist update`

However, you need to execute the below commands if you have installed:
- jBallerina 1.1.0 or 1.1.1 but switched to use a previous version: `ballerina dist pull jballerina-1.1.2`
- jBallerina 1.1.2 via the installers: `ballerina dist use jballerina-1.1.2`
- a jBallerina version below 1.1.0: install via the [installers](https://ballerina.io/downloads/)

### For new users:

If you have not installed jBallerina, then download the [installers](https://ballerina.io/downloads/) to install.

## Standard Library

- HTTP/1.1 entity trailer support.
- Request/Response header validation.

## Build Tools
- Optimize the “ballerina run” execution flow, which reduces the Ballerina program/service startup time.

## Test Framework
- Testing for individual .bal files has been implemented.
- Testerina report now lists the passed tests on the console.

## DEV Tools

### Open API
- Relative path support for OpenAPI validator.
- OpenAPI tool bug fixes.

### IDE Plugins
- Chain invocation support for the 'Create Variable' code action.
- Version support for the `Pull from Ballerina Central` code action.
- Array literal line break support for Formatting.
- Code completion and live template related fixes for the IntelliJ plugin.
- Bug fixes related to Code Formatting.
