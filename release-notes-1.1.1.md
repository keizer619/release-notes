# Overview of Ballerina 1.1.1

Ballerina 1.1.1 is a patch release iteration of its previous 1.1.0 release, which introduces the below functionalities. Also, this release addresses a few [bug fixes](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+label%3AType%2FBug+milestone%3A%22Ballerina+1.1.1%22+is%3Aclosed+)
and [improvements](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+milestone%3A%22Ballerina+1.1.1%22+is%3Aclosed+label%3AType%2FImprovement+). 

You can use the inbuilt updating capability (introduced in Ballerina 1.1.0) to update to Ballerina 1.1.1 by executing the respective command from the below list.

<table>
<tr border=1>
<th> Installed version </th>
<th> Current active version </th>
<th> Command </th>
</tr>
<tr border=1>
<td>  jBallerina-1.1.0 </td>
<td>  jBallerina-1.1.0 </td>
<td> `ballerina dist update` </td>
</tr>
<tr border=1>
<td>  jBallerina-1.1.0 </td>
<td>  versions below jBallerina-1.1.0 </td>
<td> `ballerina dist pull jballerina-1.1.1` </td>
</tr>
</table>

If you have not installed jBallerina or if you have installed a version prior to 1.1.0, then [download the installers](ballerina.io/downloads/) to install.

> **Note:** However, if you installed jBallerina via the installers when you have jBallerina 1.1.0 installed, execute the following command to activate it: `ballerina dist use jballerina-1.1.1`

## Standard Library

- HTTP2 response trailer support
- Add support for reading array values via the config-api
- Add cache for OAuth2-based inbound authentication

## Build Tools

Add major improvements to the Ballerina build phase to reduce the total build time by ~30%.

## IDE Plugins

Add support for the Signature Help feature.

## Language Specification Deviation Fixes

Previously, the implementation initialized an object field’s default value in the `__init()` function of the object, which caused the default values to be set to the field even when the `__init()` method was called explicitly. 

For example, the below sample prints “foo” as the value of `f.s` since the default value “foo” was set as the value of `f.s` whenever the `__init()` method was called. 

```ballerina
import ballerina/io;
 
type Foo object {
  
   string s = "foo";
 
   function __init() {       
   }
 
   function reInit() {
       self.__init();
   }
};
 
public function main() {
   Foo f = new();
 
   // Update field `s`.
   f.s = "bar";
 
   f.reInit();
   io:println(f.s);
}
```
This is fixed in 1.1.1 so that now this sample prints “bar”.
