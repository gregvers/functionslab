# Python Functions

This lab introduces the
[Fn Python FDK (Function Development Kit)](https://github.com/fnproject/fdk-python)
and takes you through the developer experience for building and unit testing
Python functions.

> As you make your way through this lab, look out for this icon. ![user
input](images/userinput.png) Whenever you see it, it's time for you to perform
an action.

## Creating a basic Python Function

Let's start by creating a new function.  In a terminal type the following:

![user input](images/userinput.png)
>`fn init --runtime python pythonfn`

The output will be:

```sh
Creating function at: ./pythonfn
Function boilerplate generated.
func.yaml created.
```

![user input](images/userinput.png)
>```sh
>cd pythonfn
>```

The `fn init` command creates an simple function with a bit of boilerplate to
get you started. The `--runtime` option is used to indicate that the function
we're going to develop will be written in Python 3, the default version as of
this writing. A number of other runtimes are also supported.  

Use the `find` command to see the directory structure and files that the
`init` command has created.

![user input](images/userinput.png)
>`find .`

```sh
.
./func.py
./func.yaml
./requirements.txt
```

As usual, the init command has created a `func.yaml` file for your
function but in the case of Python it also creates `requirements.txt` file
as well as a function code `func.py`.

Take a look at the contents of the generated func.yaml file.

![user input](images/userinput.png)
>```sh
>cat func.yaml
>```

```yaml
schema_version: 20180708
name: pythonfn
version: 0.0.1
runtime: python
entrypoint: /python/bin/fdk /function/func.py handler
memory: 256
```

The generated `func.yaml` file contains metadata about your function and
declares a number of properties including:

* schema_version--identifies the version of the schema for this function file
* name--the name of the function. Matches the directory name.
* version--the version of the function
* runtime--the language used for this function
* entrypoint--the name of the executable to invoke when your function is called,
  in this case `/python/bin/fdk /function/func.py handler`.

The Python function init also generates `requirements.txt` file to include any
Python libraries your function requires and that are not included with the Python
runtime. `requirements.txt` includes the Fn Python FDK runtime by default.

## Deploy your Python Function

With the `pythonfn` directory containing `requirements.txt` and `func.yaml` you've got
everything you need to deploy the function to Oracle Functions.

Make sure your context is set to point to Oracle Functions. Use the `fn list
context` command to check. If everything is fine then let's deploy the function
to the app you created in the previous lab.  It should be named `labapp-NNN`
where `NNN` is your lap participant number.

![user input](images/userinput.png)
>`fn --v deploy --app labapp-NNN `

```shell
Deploying pythonfn to app: test
Bumped to version 0.0.2
Building image phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2
FN_REGISTRY:  phx.ocir.io/mytenancy/labapp-NNN
Current Context:  ocicpm
Sending build context to Docker daemon  5.632kB
Step 1/12 : FROM fnproject/python:3.6-dev as build-stage
 ---> aa4e9945b65f
Step 2/12 : WORKDIR /function
 ---> Using cache
 ---> daec60dbb5e6
Step 3/12 : ADD requirements.txt /function/
 ---> Using cache
 ---> 0c72e8ca1470
Step 4/12 : RUN pip3 install --target /python/  --no-cache --no-cache-dir -r requirements.txt &&			 rm -fr ~/.cache/pip /tmp* requirements.txt func.yaml Dockerfile .venv
 ---> Using cache
 ---> 9164b488f7ba
Step 5/12 : ADD . /function/
 ---> 4595cc2b018d
Step 6/12 : RUN rm -fr /function/.pip_cache
 ---> Running in 47885c994366
Removing intermediate container 47885c994366
 ---> 4db7ffff2c09
Step 7/12 : FROM fnproject/python:3.6
 ---> c31b51a1f190
Step 8/12 : WORKDIR /function
 ---> Using cache
 ---> 0bb2d2798c55
Step 9/12 : COPY --from=build-stage /function /function
 ---> ee1727d317e4
Step 10/12 : COPY --from=build-stage /python /python
 ---> 0d3913d0f1f6
Step 11/12 : ENV PYTHONPATH=/python
 ---> Running in b9b0b328523b
Removing intermediate container b9b0b328523b
 ---> 895a1076b2a9
Step 12/12 : ENTRYPOINT ["/python/bin/fdk", "/function/func.py", "handler"]
 ---> Running in 8f89ffe44489
Removing intermediate container 8f89ffe44489
 ---> 2b382a1ad13e
Successfully built 2b382a1ad13e
Successfully tagged phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2

Parts:  [phx.ocir.io mytenancy labapp-NNN pythonfn:0.0.2]
Pushing phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2 to docker registry...The push refers to repository [phx.ocir.io/ocicpm/labapp-NNN/pythonfn]
436bdc86a9fd: Pushed
989f0fccd957: Pushed
3b400ebd7c53: Pushed
be88fde66395: Layer already exists
aa8525846ddf: Layer already exists
808c4a375127: Layer already exists
f1e324b9134c: Layer already exists
0b776d2c2318: Layer already exists
5dacd731af1b: Layer already exists
0.0.2: digest: sha256:47e6c650a0b615353efb05837d560efc248bb9d7bf385a784ce7543479f6b42d size: 2207
Updating function pythonfn using image phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2...
Successfully created function: pythonfn with phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2
```

The output message
`Updating function pythonfn using image phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2...`
let's us know that the function is packaged in the image
"phx.ocir.io/mytenancy/labapp-NNN/pythonfn:0.0.2".

## Invoke your Deployed Function

Use the the `fn invoke` command to call your function from the command line.

### Invoke with the CLI

The first is using the Fn CLI which makes invoking your function relatively
easy.  Type the following:

![user input](images/userinput.png)
>```sh
> fn invoke labapp-NNN pythonfn
>```

which results in:

```json
{"message": "Hello World"}
```

You can also pass data to the invoke command. For example:

![user input](images/userinput.png)
>```sh
> echo -n '{"name": "Bob"}' | fn invoke test pythonfn
>```

```json
{"message": "Hello Bob"}
```

"Bob" was passed to the function where it is processed and returned in the
output.

## Exploring the Code

We've generated, compiled, deployed, and invoked the Python function so let's take
a look at the code.  You may want to open the code in one of the IDEs available
in the lab environment.

Below is the generated Python function.  As you can see the function takes a
JSON text in input and returns another JSON text.

```python
import io
import json

from fdk import response

def handler(ctx, data: io.BytesIO=None):
    name = "World"
    try:
        body = json.loads(data.getvalue())
        name = body.get("name")
    except (Exception, ValueError) as ex:
        print(str(ex))

    return response.Response(
        ctx, response_data=json.dumps(
            {"message": "Hello {0}".format(name)}),
        headers={"Content-Type": "application/json"}
    )
```

This function returns the JSON text {"message": "Hello World"} unless an input
is provided, in which case it returns {"message": "Hello &lt;name&gt;"}.  We saw
this previously when we piped {"name": "Bob"} into the function.   Notice that
the Python FDK reads JSON data from standard input that can be converted into a
dictionary.

## Testing with Pytest

The `fn init` command also generated a JUnit test for the function which uses
the Java FDK's function test framework.  With this framework you can setup test
fixtures with various function input values and verify the results.

The Python FDK supports Pytest testing framework that allows performing unit tests
of your function's code. Here is an example:

![user input](images/userinput.png) Create a python file named `test_func.py`
with the following content and place is in your function directory.

```python
import io
import json
import pytest

from fdk import fixtures
from func import handler

@pytest.mark.asyncio
async def test_func_without_data():
    call = await fixtures.setup_fn_call(handler)
    content, status, headers = await call
    assert 200 == status
    assert {"message": "Hello World"} == json.loads(content)
    assert "application/json" == headers.get("content-type")
```

The generated test confirms that when no input is provided the function returns
{"message": "Hello World"}.

Let's add a test that confirms that when an input string like {"name": "Bob"} is
provided we get the expected result.

![user input](images/userinput.png) Add the following block to `test_func.py` file:

```python
@pytest.mark.asyncio
async def test_func_with_data():
    input = io.BytesIO(b'{"name": "Bob"}')
    call = await fixtures.setup_fn_call(handler, content=input)
    content, status, headers = await call
    assert 200 == status
    assert {"message": "Hello Bob"} == json.loads(content)
    assert "application/json" == headers.get("content-type")
```

You can see the content argument for setup_fn_call is used to specify the value
of the function's input.

You can run the tests by running from the function's directory:
```shell
pytest -v -s --tb=long
```

## Wrap Up

Congratulations! You've just completed an introduction to the Fn Python FDK.
There's so much more in the FDK than we can cover in a brief introduction.  Next
we'll take a look at troubleshooting.  We saw a little of this when our tests
failed but there are a few more techniques available.

NEXT: [*Troubleshooting*](5-Troubleshooting.md), UP: [*Labs*](1-Labs.md), HOME:
[*INDEX*](README.md)
