
# Overview to Ballerina 0.970.0-beta1
Overview to Ballerina 0.970.0-beta6
This release include an improvements to the standard library APIs. The full list of APIs have been reviewed  and enhancements incorporated to this release to make the API usage simpler to the developer. 

For example the changes to the signatures of the HTTP client functions for the standard HTTP methods, to make the Request parameter, makes the usage much simple for developer. 
```ballerina
http:Request req = new;
var clientResponse = clientEP->get("/echo"); // Passing a request is optional
var clientResponse = clientEP->get("/echo", request = req); 
```
Another example of the API refactoring from HTTP package is the merge of both `Client` and SimpleClient into a single endpoint called `Client`. 
The simplified use of a sample of `FailoverClient` configuration:
```ballerina
endpoint http:FailoverClient ClientEP {
    timeoutMillis:5000,
    failoverCodes:[501, 502, 503],
    intervalMillis:5000,
    targets:[
        {
            url:"http://localhost:3000/mock1"
        },
        {
            url:"http://localhost:8080/echo",
            secureSocket:{
                trustStore:{
                    filePath:"${ballerina.home}/bre/security/ballerinaTruststore.p12",
                    password:"ballerina"
                },
                verifyHostname:false,
                shareSession:true
            }
        },
        {
            url:"http://localhost:8080/mock2"
        }
    ]
};
```
A sample of `Client` configuration:
```ballering
endpoint http:Client clientEP {
    url:"http://localhost:8080/mock",
    timeoutMillis:5000
};
```
Numerous similar refactoring has been done across the standard library API and you can find further details by referring the latest API documentation. 


Compiled version of a Ballerina package support, in short form referred to as “balo” in developer community, has been added to this release. 
When packages are built into a .balo, they will be installed into the project repository. 
```
ballerina build -c <package> [-o <output-file>.balo]
```

This release ships many stabilization enhancements to the type system elements that were introduced in 0.970.0-beta0. All other parts of the language implementation has added many fixes. 

# Compatibility and Support
You will have to update the API usage in sync with the API enhancements. Please use the latest API documentation to explore the latest API. 

# Improvements
## Language & Runtime
- Standard library API enhancements and refactoring 
- Compiled version of a Ballerina package (balo) support

# Getting Started
You can download the Ballerina distributions, try samples, and read the documentation at <https://ballerina.io>. You can also visit [Quick Tour][1] to get started. We encourage you to report issues, improvements, and suggestions at [Ballerina Github Repository][2].

[1]: https://ballerina.io/learn/quick-tour/
[2]: https://github.com/ballerina-platform/ballerina-lang
