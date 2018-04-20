# Overview to Ballerina 0.970.0-beta1
This release has improvements to the main method argument passing model where the user will not have to wrestle with string array to access command line parameters.

Auto completion support is added in language server / IDE for match statements to help unions to be matched and tuples have to be destructured.

This release also introduces the message broker. Ballerina Message Broker is a lightweight, easy-to-use, message-brokering server. It uses AMQP 0-9-1 as the messaging protocol. This release embeds the message broker into the Ballerina platform distribution.

Enhancements to manage configurations and anonymous endpoints for Kubernetes and Docker has been introduced in this release.

Apart from these enhancements, this release ships many stabilization enhancements on top of the previous release, which was 0.970.0-beta0.

# Compatibility and Support
There are no compatibility and support changes from 0.970.0-beta0 other than the syntax change of main (string... args)

# Improvements
## Language & Runtime
- Main method argument parameters has been changed as main (string... args)

## IDEs & Language Server
- “match” expression completion support in Language server

## Ballerina Message Broker
- Include message broker inside Ballerina platform distribution and should be able to start the broker from the bin

## Ballerina Extensions
- Added copy file support to Kubernetes and Docker images
- Added first class support for BallerinaConf and remove name, mount path request from the user
- Add anonymous endpoint support for Kubernetes and Docker

# Getting Started
You can download the Ballerina distributions, try samples, and read the documentation at http://ballerinalang.org. You can also visit [Quick Tour][1] to get started. We encourage you to report issues, improvements, and suggestions at [Ballerina Github Repository][2].

[1]: https://ballerinalang.org/docs/quick-tour/quick-tour
[2]: https://github.com/ballerina-lang/ballerina
