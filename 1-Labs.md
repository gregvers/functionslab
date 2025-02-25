# Functions Labs

In each of the following labs you'll explore a different aspect of Oracle
Functions from creating, to deploying, to troubleshooting, to invoking.  In all
cases you'll be using the `fn` CLI extensively so you may find these references
helpful:

* [fn Commands
  Cheatsheet](https://github.com/sachin-pikle/functionslab/wiki/Functions-Commands-Cheatsheet)

* [fn CLI docs](https://github.com/fnproject/docs/blob/master/cli/README.md)

## Your First Function

Now that `fn` CLI is installed and your development environment is configured,
we can dig into creating and running functions. Fn provides a FDK
(Function Development Kit) for each of the core supported programming languages.
In this lab you'll create, deploy, and run your first function.
* [First function with Java](3a-First-Function-java.md)
* [First function with Python](3b-First-Function-python.md)
* [First function with Nodejs](3c-First-Function-nodejs.md)

## Unit Testing

The FDK supports Unit Testing for the core supported programming languages.
* [Unit Testing with Java](4a-Unit-Testing-java.md)
* [Unit Testing with Python](4b-Unit-Testing-python.md)

## Troubleshooting

If you've been following the instructions in the tutorials carefully you
shouldn't have run into any unexpected failures--hopefully!!  But in real life
when you're writing code things go wrong--builds fail, exceptions are thrown,
etc.  Fortunately the
[Troubleshooting](5-Troubleshooting.md) tutorial
introduces techniques you can use to track down the source of a failure.

## Automatically invoke Functions using OCI Events service

OCI Events service lets you monitor OCI resource changes and send
notifications and/or trigger a function automatically in response to that
change. We'll explore sending email notifications, and invoking a function via
the OCI Events when an image is uploaded to an OCI Object Storage bucket in
[Automatically invoke Functions using OCI Events](9-Functions-Invoke-OCI-Events.md).

## Functions Clients

Functions can be invoked over HTTP using their "invoke endpoint". You can
either invoke the endpoint directly or use the OCI SDK to both manage and
invoke functions. We'll explore how to invoke a function using:
* [OCI SDK for Functions](8-Functions-Clients-SDK.md)
* [`oci-curl` utility](7-Functions-Clients-oci-curl.md)

## Containers as Functions

One of the coolest features of Fn is that while it's easy to write functions in
various programming languages, you can also deploy Docker images as functions.
This opens up entire worlds of opportunity as you can package existing code,
utilities, or use a programming language not yet supported by Fn.  
Try the tutorial to see how easy it is:
* [Containers as Functions with Nodejs](6c-Container-as-Function-nodejs.md)
* [Containers as Functions with Python](6b-Container-as-Function-python.md)
