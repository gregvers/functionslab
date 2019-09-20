# Your First Function with Java

In this introductory lab we'll walk through developing a function using the
Java programming language and deploying that function with Oracle Functions.  We'll
also learn about the core Fn concepts like applications and triggers.

> As you make your way through this lab, look out for this icon.
![user input](images/userinput.png) Whenever you see it, it's time for you to
perform an action.

Let's start with a very simple "hello world" function written in [Java]
(https://www.oracle.com/java/). Don't worry, you don't need to know Java!  In
fact you don't even need to have the Java JDK or JRE installed on your development machine as
Fn provides the necessary Java tools as a Docker container.  Let's walk through
your first function to become familiar with the process and how Fn supports
development.

## Create your Function
In the terminal type the following.

![user input](images/userinput.png)
>```
> fn init --runtime python javafn
>```

The output will be:
```sh
Creating function at: /javafn
Function boilerplate generated.
func.yaml created.
```

The `fn init` command creates an simple function with a bit of boilerplate to
get you started. The `--runtime` option is used to indicate that the function
we're going to develop will be written in Java 11, the default version as of
this writing. A number of other runtimes are also supported.  

### Review the generated files

With your function created change into the `/javafn` directory.

![user input](images/userinput.png)
>```sh
>cd javafn
>```

Use the `find` command to see the directory structure and files that the
`init` command has created.

![user input](images/userinput.png)
>`find .`

```sh
.
./func.yaml
./pom.xml
./src
./src/test
./src/test/java
./src/test/java/com
./src/test/java/com/example
./src/test/java/com/example/fn
./src/test/java/com/example/fn/HelloFunctionTest.java
./src/main
./src/main/java
./src/main/java/com
./src/main/java/com/example
./src/main/java/com/example/fn
./src/main/java/com/example/fn/HelloFunction.java
```

The init command has created a `func.yaml` file for your
function but in the case of Java it also creates a Maven `pom.xml` file
as well as a function class and function test class.

### Exploring the Function Code
with several supporting files. You may want to open the
`com.example.fn.HelloFunction` class file in one of the IDEs available in the
lab environment or type:

![user input](images/userinput.png)
>```sh
> cat ./src/main/java/com/example/fn/HelloFunction.java
>```

```java
package com.example.fn;

public class HelloFunction {

    public String handleRequest(String input) {
        String name = (input == null || input.isEmpty()) ? "world"  : input;

        return "Hello, " + name + "!";
    }

}
```
As you can see the function is just a method on a POJO that takes a string value
and returns another string value, but the Java FDK also supports binding
input parameters to streams, primitive types, byte arrays and Java POJOs
unmarshalled from JSON.  Functions can also be static or instance
methods.

This function returns the string "Hello, world!" unless an input string
is provided, in which case it returns "Hello, &lt;input string&gt;!". Notice that
the Java FDK reads from standard input and automatically puts the
content into the string passed to the function.  This greatly simplifies
the function code.

### Understanding func.yaml

The `fn init` command generated a `func.yaml` function
configuration file. Let's look at the contents:

![user input](images/userinput.png)
>```sh
> cat func.yaml
>```

```yaml
schema_version: 20180708
name: javafn
version: 0.0.1
runtime: java
build_image: fnproject/fn-java-fdk-build:jdk11-1.0.86
run_image: fnproject/fn-java-fdk:jre11-1.0.86
cmd: com.example.fn.HelloFunction::handleRequest
```

The generated `func.yaml` file contains metadata about your function and
declares a number of properties including:

* schema_version--identifies the version of the schema for this function file
* version--the version of the function
* runtime--the language used for this function
* build_image--the image used to build your function's image
* run_image--the image your function runs in
* cmd--the `cmd` property is set to the fully qualified name of the Java
  class and method that should be invoked when your `javafn` function is
  called

### The pom.xml file

The Java function init also generates a Maven `pom.xml` file to build and test
your function.  The pom includes the Fn Java FDK runtime and the test libraries
your function needs.

## Your Deployment Target

With the `javafn` directory containing a Java class file, `func.yaml` and pom.xml
you've got everything you need to deploy the function to Oracle Functions.

Make sure your context is set to point to Oracle Functions. Use the `fn list
contexts` command to check.

![user input](images/userinput.png)
>```sh
> fn list contexts
>```

The `API URL` and `REGISTRY` values of the row with the `*` should be pointing
to Oracle Functions and OCIR, respectively:

```shell
CURRENT    NAME        PROVIDER    API URL                                           REGISTRY
           default     default                                                       gregvers
*          workshop    oracle      https://functions.us-phoenix-1.oraclecloud.com    phx.ocir.io/mytenancy/myuser
```

If your context *is not* configured correctly, please return to the
[*Environment Setup*](0-Setup.md) instructions before proceeding.


## Deploying your Function

Deploying your function is how you publish the function and make it accessible
to other users and systems. To see the details of what is happening during a
function deploy,  use the `-v / --verbose` switch.  The first time you build a
function of a particular language it takes longer as the `fn` CLI must download
the necessary Docker images. The `--verbose` option allows you to see this
process.

### Creating an Application

Before you can deploy a function you'll need to create an application.  You can
do this using the `fn` CLI or in the Oracle Functions web console.  We'll use the
console as it's somewhat simpler to click on options than to copy/paste network
IDs for use on the command line.

Open your browser to the Oracle Functions console and
login:

![user input](images/userinput.png) https://console.us-ashburn-1.oraclecloud.com/functions

![user input](images/userinput.png) Once you've logged in, select the Phoenix (us-phoenix-1) region:

![Region Selection](images/region-selection.png)

![user input](images/userinput.png) Navigate down the compartment hierarchy on the left hand dropdown to the compartment your instructor will provide.

![Applications List](images/applications-compartment.png)

If you haven't selected the `us-phoenix-1` region you'll see the following error.  To
correct simply choose the Phoenix region from the drop down menu.

![Functions Unavailable](images/functions-unavailable.png)

![user input](images/userinput.png) click "Create Application" and complete the
form with following values where NNN is your lab participant number.  

**IMPORTANT NOTE**: Lab participants are all working in the same OCI tenancy and
compartment so to avoid confusion you need to name your applications with your
participant number. Wherever you see `NNN` in the lab instructions please
substitute in your number.

>```sh
> name: `labapp-NNN`
> vcn: `workshop-vcn`
> subnet: `Public Subnet nFuS:PHX-AD-1`
>```

![Create Application](images/create-application.png)

Functions deployed as part of this application will be attached to the
specifiied vcn and subnet.

![user input](images/userinput.png) click "Create" to finish.

### Building and Deploying your Function

The `fn` CLI makes it easy to build and deploy functions.  Building a function
is the process of compiling (if necessary) and packaging your code into a Docker
container.  The deployment step will push the container to OCIR and define the
function in Oracle Functions.  You can do all of this in one command.

![user input](images/userinput.png)
>```sh
> fn --v deploy --app labapp-NNN
>```

When we deploy a single function we have to specify the application it belongs
to.

You should see output similar to:

```shell
Deploying javafn to app: labapp-NNN
Building image phx.ocir.io/mytenancy/myuser/javafn:0.0.2
FN_REGISTRY:  phx.ocir.io/mytenancy/myuser
Current Context:  workshop
Sending build context to Docker daemon  14.34kB
Step 1/11 : FROM fnproject/fn-java-fdk-build:jdk11-1.0.86 as build-stage
 ---> 0ab30d8e3524
Step 2/11 : WORKDIR /function
 ---> Using cache
 ---> d6c3e60e0c04
Step 3/11 : ENV MAVEN_OPTS -Dhttp.proxyHost= -Dhttp.proxyPort= -Dhttps.proxyHost= -Dhttps.proxyPort= -Dhttp.nonProxyHosts= -Dmaven.repo.local=/usr/share/maven/ref/repository
 ---> Using cache
 ---> 9133a74699d5
Step 4/11 : ADD pom.xml /function/pom.xml
 ---> Using cache
 ---> f41ec165b9a8
Step 5/11 : RUN ["mvn", "package", "dependency:copy-dependencies", "-DincludeScope=runtime", "-DskipTests=true", "-Dmdep.prependGroupId=true", "-DoutputDirectory=target", "--fail-never"]
 ---> Using cache
 ---> 384bcd123a67
Step 6/11 : ADD src /function/src
 ---> Using cache
 ---> 2f3afddbba1b
Step 7/11 : RUN ["mvn", "package"]
 ---> Using cache
 ---> 00a1b46e3258
Step 8/11 : FROM fnproject/fn-java-fdk:jre11-1.0.86
 ---> d7caad608803
Step 9/11 : WORKDIR /function
 ---> Using cache
 ---> d075b963bbfd
Step 10/11 : COPY --from=build-stage /function/target/*.jar /function/app/
 ---> Using cache
 ---> c6a836a20c57
Step 11/11 : CMD ["com.example.fn.HelloFunction::handleRequest"]
 ---> Using cache
 ---> 10586c295622
Successfully built 10586c295622
Successfully tagged phx.ocir.io/mytenancy/myuser/javafn:0.0.2

Parts:  [phx.ocir.io mytenancy myuser javafn:0.0.2]
Pushing phx.ocir.io/mytenancy/myuser/javafn:0.0.2 to docker registry...The push refers to repository [phx.ocir.io/mytenancy/myuser/javafn]
...
Updating function javafn using image phx.ocir.io/mytenancy/myuser/javafn:0.0.2...
```

Since we turned on verbose mode, the steps to build the Docker container image
are displayed. Normally you deploy an application without the `-v/--verbose`
option. If you rerun the command a new image and version is created, pushed to
OCIR, and deployed.

The output message
`Updating function javafn using image phx.ocir.io/mytenancy/myuser/javafn:0.0.2...`
let's us know that the function is packaged in the image
"phx.ocir.io/mytenancy/myuser/javafn:0.0.2".

### Functions in the Oracle Functions Console

Click on `labapp-NNN` in the Functions Applications List and you'll see the
`pythonfn` function appears in the functions list.

![labapp-NNN Functions](images/labapp-pythonfn.png)

## Invoking your Deployed Function

There are a few ways you can invoke your function.  The easiest is with the `fn`
CLI.  We'll see other ways in subsequent labs. Type the following:

![user input](images/userinput.png)
>```sh
> fn invoke labapp-NNN javafn
>```

which results in:

```txt
Hello, World!
```

You can also pass data to the invoke command. For example:

![user input](images/userinput.png)
>```sh
> echo -n 'Bob' | fn invoke labapp-NNN javafn
>```

```txt
Hello, Bob!
```

"Bob" was passed to the function where it is processed and returned in the
output.

The first time you call a function in a new application you may encounter what
is commonly called a "cold start," which results in the function call taking
longer than usual. When you invoked "labapp-NNN pythonfn," Oracle Functions looked
up the "labapp-NNN" application and then looked for the Docker container image
bound to the "pythonfn" function and executed the code. The necessary compute
infrastructure was also allocated.  When you invoke the function a second time
you'll notice it's much faster.

### Understand fn deploy

If you have used Docker before the output of `fn --verbose deploy` should look
familiar--it looks like the output you see when running `docker build` with a
Dockerfile.  Of course this is exactly what's happening!  When you deploy a
function like this the `fn` CLI is dynamically generating a Dockerfile for your
function and building a container image.

> __NOTE__: two images are actually being used.  The first contains the language
> compiler and all the necessary build tools. The second image packages all
> dependencies and any necessary language runtime components. Using this
> strategy, the final function image size can be kept as small and secure as
> possible. Smaller Docker images are naturally faster to push and pull from a
> registry which improves overall performance.  For more details on this
> technique see [Multi-Stage Docker Builds for Creating Tiny Go
> Images](https://medium.com/travis-on-docker/multi-stage-docker-builds-for-creating-tiny-go-images-e0e1867efe5a).

As the `fn` CLI is built on Docker you can use the `docker` command to see the local
container image you just generated. You may have a number of Docker images so
use the following command to see only versions of javafn:

![user input](images/userinput.png)
>```sh
> docker images | grep javafn
>```

You should see something like:

```sh
phx.ocir.io/mytenancy/myuser/javafn   0.0.2    5c0b79de04e8   15 minutes ago  66.3MB
phx.ocir.io/mytenancy/myuser/javafn   0.0.1    4936dbed0df6   40 minutes ago  66.3MB
```

### Explore your Application

The fn CLI provides a couple of commands to let us see what we've deployed.
`fn list apps` returns a list of all of the defined applications.

![user input](images/userinput.png)
>```sh
> fn list apps
>```

Which, in our case, returns the name of the application we created when we
deployed our `pythonfn` function:

```shell
NAME         ID
labapp-NNN   ocid1.fnapp.oc1.us-phoenix-1.aaaaaaaaafe33ayhrq5dkeplxk6wd5fpyhc7chufehxjhv7kgbnrbevbovma
```

We can also see the functions that are defined by an application. To list the
functions included in "labapp-NNN" we can type:

![user input](images/userinput.png)
>```sh
> fn ls f labapp-NNN
>```

```sh
NAME    IMAGE                                                  ID
javafn  phx.ocir.io/mytenancy/myuser/pythonfn:0.0.2    ocid1.fnfunc.oc1.us-phoenix-1.aaaaaaaaacm4u6futn2q34fh4hbkirwsxdffss42kvd3kl6xxakkful5yehq
```

## Wrap Up

Congratulations!  In this lab you've accomplished a lot.  You've created
your first function, deployed it to Oracle Functions and invoked it!

NEXT: [*Unit Testing with Java*](4a-JUnit-Testing-java.md), UP: [*Labs*](1-Labs.md), HOME:
[*INDEX*](README.md)
