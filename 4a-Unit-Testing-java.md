# Testing Java Functions with JUnit

The [Fn Java FDK (Function Development Kit)](https://github.com/fnproject/fdk-java)
provides you with the developer experience for building and unit testing
Java functions.

The `fn init` command generates a JUnit test for the function which uses
the Java FDK's function test framework.  With this framework you can setup test
fixtures with various function input values and verify the results.

The generated test confirms that when no input is provided the function returns
"Hello, world!".

```java
package com.example.fn;

import com.fnproject.fn.testing.*;
import org.junit.*;

import static org.junit.Assert.*;

public class HelloFunctionTest {

    @Rule
    public final FnTestingRule testing = FnTestingRule.createDefault();

    @Test
    public void shouldReturnGreeting() {
        testing.givenEvent().enqueue();
        testing.thenRun(HelloFunction.class, "handleRequest");

        FnResult result = testing.getOnlyResult();
        assertEquals("Hello, world!", result.getBodyAsString());
    }

}
```

Let's add a test that confirms that when an input string like "Bob" is
provided we get the expected result.

![user input](images/userinput.png) Add the following method to the `HelloFunctionTest` class:


```java
    @Test
    public void shouldReturnWithInput() {
        testing.givenEvent().withBody("Bob").enqueue();
        testing.thenRun(HelloFunction.class, "handleRequest");

        FnResult result = testing.getOnlyResult();
        assertEquals("Hello, Bob!", result.getBodyAsString());
    }
```

You can see the `withBody()` method used to specify the value of the
function input.

You can run the tests by building your function with `fn build`.  This will
cause Maven to compile and run the updated test class.  You can also invoke your
tests directly from Maven using `mvn test` or from your IDE.

![user input](images/userinput.png)
>`fn build`

```sh
Building image fndemouser/javafn:0.0.2 .......
Function fndemouser/javafn:0.0.2 built successfully.
```

## Accepting JSON Input

Let's convert this function to use JSON for its input and output.

![user input](images/userinput.png) Replace the definition of `HelloFunction`
with the following:

```java
package com.example.fn;

public class HelloFunction {

    public static class Input {
        public String name;
    }

    public static class Result {
        public String salutation;
    }

    public Result handleRequest(Input input) {
        Result result = new Result();
        result.salutation = "Hello " + input.name;
        return result;
    }

}
```

We've created a couple of simple Pojos to bind the JSON input and output
to and changed the function signature to use these Pojos.  The
Java FDK will *automatically* bind input data based on the Java arguments
to the function. JSON support is built-in but input and output binding
is extensible and you could plug in marshallers for other
data formats like protobuf, avro or xml.

Let's build the updated function:

![user input](images/userinput.png)
>`fn build`

returns:

```sh
Building image fndemouser/javafn:0.0.2 .....
Error during build. Run with `--verbose` flag to see what went wrong. eg: `fn --verbose CMD`

Fn: error running docker build: exit status 1

See 'fn <command> --help' for more information. Client version: 0.5.16
```

Uh oh! To find out what happened rerun build with the verbose switch:

![user input](images/userinput.png)
>`fn --verbose build`

```sh
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.example.fn.HelloFunctionTest
An exception was thrown during Input Coercion: Failed to coerce event to user function parameter type class com.example.fn.HelloFunction$Input
...
An exception was thrown during Input Coercion: Failed to coerce event to user function parameter type class com.example.fn.HelloFunction$Input
...
Tests run: 2, Failures: 0, Errors: 2, Skipped: 0, Time elapsed: 0.893 sec <<< FAILURE!
...
Results :

Tests in error:
  shouldReturnGreeting(com.example.fn.HelloFunctionTest): One and only one response expected, but 0 responses were generated.
  shouldReturnWithInput(com.example.fn.HelloFunctionTest): One and only one response expected, but 0 responses were generated.

Tests run: 2, Failures: 0, Errors: 2, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.477 s
[INFO] Finished at: 2017-09-21T14:59:21Z
[INFO] Final Memory: 16M/128M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test (default-test) on project hello: There are test failures.
```

*Oops!* as we can see this function build has failed due to test failures--we
changed the code significantly but didn't update our tests!  We really
should be doing test driven development and updating the test first, but
at least our bad behavior has been caught.  Let's update the tests
to reflect our new expected results.  

![user input](images/userinput.png) Replace the definition of
`HelloFunctionTest` with:

```java

package com.example.fn;

import com.fnproject.fn.testing.*;
import org.junit.*;

import static org.junit.Assert.*;

public class HelloFunctionTest {

    @Rule
    public final FnTestingRule testing = FnTestingRule.createDefault();

    @Test
    public void shouldReturnGreeting(){
        testing.givenEvent().withBody("{\"name\":\"Bob\"}").enqueue();
        testing.thenRun(HelloFunction.class,"handleRequest");

        FnResult result = testing.getOnlyResult();
        assertEquals("{\"salutation\":\"Hello Bob\"}", result.getBodyAsString());
    }
}

```

In the new `shouldReturnGreeting()` test method we're passing in the
JSON document

```js
{
    "name": "Bob"
}
```
and expecting a result of
```js
{
    "salutation": "Hello Bob"
}
```

If you re-run the tests in your IDE or via `fn -verbose build` we can see that
it now passes:

![user input](images/userinput.png)
>`fn --verbose build`

## Wrap Up

Congratulations! You've just completed an introduction to the Fn Java FDK.
There's so much more in the FDK than we can cover in a brief introduction.  Next
we'll take a look at troubleshooting.  We saw a little of this when our tests
failed but there are a few more techniques available.

NEXT: [*Troubleshooting*](5-Troubleshooting.md), UP: [*Labs*](1-Labs.md), HOME:
[*INDEX*](README.md)
