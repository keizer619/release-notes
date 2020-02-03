# Overview of Ballerina 1.1.1

Ballerina 1.1.1 is a patch release iteration of its previous 1.1.0 release, which introduces the below functionalities. Also, this release addresses a few [bug fixes](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+label%3AType%2FBug+milestone%3A%22Ballerina+1.1.1%22+is%3Aclosed+)
and [improvements](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+milestone%3A%22Ballerina+1.1.1%22+is%3Aclosed+label%3AType%2FImprovement+). 

## Ballerina Update Tool

The inbuilt updating capability (i.e., the Ballerina Update tool) introduced in Ballerina 1.1.0 is improved with the following usages. Now, you can update Ballerina by executing the below commands based on the Ballerina and JBallerina versions that you have installed.

<table border=1>
<tr>
<th> Command </th>
<th> Usage </th>
<tr>
<td> `ballerina dist update` </td>
<td>  Ballerina 1.1.0 and JBallerina-1.1.0 </td>
</tr>
<tr>
<td> `ballerina dist pull jballerina-1.1.1` </td>
<td>  Ballerina 1.1.0 and previous jBallerina versions like jballerina-1.0.5 </td>
</tr>
<tr>
<td> `ballerina dist use jballerina-1.1.1` </td>
<td>  Ballerina 1.1.1 (to manually switch to 1.1.1 if installed via the installers) </td>
</tr>
</table>

If you have not installed Ballerina or if you have installed a version prior to 1.1.0, then download the installers from [download page](ballerina.io/downloads/).

## Standard Library

- HTTP2 response trailer support
- Add support for reading array values via the config-api
- Add cache for OAuth2-based inbound authentication

## Build Tools

Add major improvements to the Ballerina build phase to reduce the total build time by ~30%.

## IDE Plugins

Add support for the Signature Help feature.