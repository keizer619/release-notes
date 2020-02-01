# Overview to Ballerina 1.1.1

Ballerina 1.1.1 is a patch release iteration of its previous 1.1.0 release. Please refer to the Github repository for a list of [bug fixes](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+label%3AType%2FBug+milestone%3A%22Ballerina+1.1.1%22+is%3Aclosed+)
and [improvements](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+milestone%3A%22Ballerina+1.1.1%22+is%3Aclosed+label%3AType%2FImprovement+).

## How to update

Ballerina 1.1.0 has introduced an inbuilt updating capability into it. Therefore, if you have installed Ballerina 1.1.0 and if you are using jballerina-1.1.0, you can simply run the following command to update Ballerina.

`ballerina dist update`

If you have installed Ballerina 1.1.0 and have switched to previous jBallerina versions like jballerina-1.0.5, then you can directly fetch this release by executing the pull command as follows.

`ballerina dist pull jballerina-1.1.1`

For further details, see [How to Keep Ballerina up to date article](https://ballerina.io/learn/how-to-keep-ballerina-up-to-date/)

If you have not installed Ballerina or if you have installed a version prior to 1.1.0, then download the installers from [download page](ballerina.io/downloads/). If you have installed 1.1.1 on top of 1.1.0 using installers you need to manually switch to 1.1.1 by executing the following command

`ballerina dist use jballerina-1.1.1`

## Standard Library

- HTTP2 response trailer support
- Add support for reading array values via config-api
- Add cache for OAuth2 inbound authentication
