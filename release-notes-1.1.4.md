# Overview of jBallerina 1.1.4

Ballerina 1.1.4 is a patch release iteration of its previous 1.1.3 release. Please refer to the Github repository for
 a list of [bug fixes](https://github.com/ballerina-platform/ballerina-lang/issues?utf8=✓&q=is%3Aissue+label%3AType%2FBug+milestone%3A%22Ballerina+1.1.4%22+is%3Aclosed+) and 
 [improvements](https://github.com/ballerina-platform/ballerina-lang/issues?q=is%3Aissue+is%3Aclosed+label%3AType%2FImprovement+milestone%3A%22Ballerina+1.1.4%22).

You can use the inbuilt updating capability (introduced in jBallerina 1.1.0) to update to jBallerina 1.1.4 as follows.

### For existing users:

If you are already using **jBallerina version 1.1.0 or above**, you can directly update your distribution to jBallerina
 1.1.4 by executing the below command:
> `ballerina dist update`

However, you need to execute the below commands if you have installed:
- jBallerina 1.1.0 or above but switched to use a previous version: `ballerina dist pull jballerina-1.1.4`
- jBallerina 1.1.4 via the installers: `ballerina dist use jballerina-1.1.4`
- a jBallerina version below 1.1.0: install via the [installers](https://ballerina.io/downloads/)

### For new users:
If you have not installed jBallerina, then download the [installers](https://ballerina.io/downloads/) to install.


## Standard Library
- Added path component to file:FileInfo. This will give the absolute path of the file. 
- Handled the NullpointerException while generating the JWT key.

## Deployment
- Support to generate knative service artifacts with @knative annotation.

## DEV Tools
### Language Server
- Improvements and bug fixes to Auto-completion engine

### IntelliJ Plugin
- Add support to view language server trace logs.
- Improve handling ballerina source files inside non-ballerina projects.
- Add minor improvements to plugin settings and language server debug log view.
