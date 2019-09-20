# Your First Function with Python

In this introductory lab we'll walk through developing a function using the
Python programming language and deploying that function with Oracle Functions.  We'll
also learn about the core Fn concepts like applications and triggers.

> As you make your way through this lab, look out for this icon.
![user input](images/userinput.png) Whenever you see it, it's time for you to
perform an action.

Let's start with a very simple "hello world" function written in [Python]
(https://www.python.org/). Don't worry, you don't need to know Python!  In
fact you don't even need to have Python installed on your development machine as
Fn provides the necessary Python tools as a Docker container.  Let's walk through
your first function to become familiar with the process and how Fn supports
development.


## Create your Function
In the terminal type the following.

![user input](images/userinput.png)
>```
> fn init --runtime python pythonfn
>```

The output will be:
```sh
Creating function at: ./pythonfn
Function boilerplate generated.
func.yaml created.
```

The `fn init` command creates a simple function with a bit of boilerplate to get
you started. The `--runtime` option is used to indicate that the function we're
going to develop will be written in Python. A number of other runtimes are also
supported.  Fn creates the simple function along with several supporting files
in the `pythonfn` directory.

### Review the generated files

With your function created change into the `/pythonfn` directory.

![user input](images/userinput.png)
>```sh
> cd pythonfn
>```

Now get a list of the directory contents.

![user input](images/userinput.png)
>```sh
> ls
>```

```sh
func.py func.yaml requirements.txt
```

### Exploring the Function Code
The `func.py` file which contains your actual Python function is generated along
with several supporting files. To view your Python function type:

![user input](images/userinput.png)
>```sh
> cat func.py
>```

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

This function looks for JSON input in the form of `{"name": "Bob"}`. If this
JSON example is passed to the function, the function returns `{"message":"Hello
Bob"}`. If no JSON data is found, the function returns `{"message":"Hello
World"}`.

### Understanding func.yaml

The `fn init` command generated a `func.yaml` function
configuration file. Let's look at the contents:

![user input](images/userinput.png)
>```sh
> cat func.yaml
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

* schema_version--identifies the version of the schema for this function file.
  Essentially, it determines which fields are present in `func.yaml`.
* name--the name of the function. Matches the directory name.
* version--automatically starting at 0.0.1.
* runtime--the name of the runtime/language which was set based on the value set
  in `--runtime`.
* entrypoint--the name of the executable to invoke when your function is called,
  in this case `/python/bin/fdk /function/func.py handler`.

There are other user-specifiable properties but these will suffice for this
example.  Note that if not specified, the name of your function will be taken
from the containing folder name.

### The requirements.txt file

The `fn init` command generated one other file: `requirements.txt`. This file
specifies all the Python libraries dependencies for your Python function.
By default, `requirements.txt` contains `fdk` but you can also add any Python
libraries your code requires.

## Your Deployment Target

With the `pythonfn` directory containing `func.py`, `func.yaml` and
`requirements.txt` you've got everything you need to deploy the function to
Oracle Functions.

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

## Deploying your First Function

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

Since we turned on verbose mode, the steps to build the Docker container image
are displayed. Normally you deploy an application without the `-v/--verbose`
option. If you rerun the command a new image and version is created, pushed to
OCIR, and deployed.

The output message
`Updating function pythonfn using image phx.ocir.io/mytenancy/myuser/pythonfn:0.0.2...`
let's us know that the function is packaged in the image
"phx.ocir.io/mytenancy/myuser/pythonfn:0.0.2".

### Functions in the Oracle Functions Console

Click on `labapp-NNN` in the Functions Applications List and you'll see the
`pythonfn` function appears in the functions list.

![labapp-NNN Functions](images/labapp-pythonfn.png)

## Invoke your Deployed Function

There are a few ways you can invoke your function.  The easiest is with the `fn`
CLI.  We'll see other ways in subsequent labs. Type the following:

![user input](images/userinput.png)
>```sh
> fn invoke labapp-NNN pythonfn
>```

which results in:

```python
{"message":"Hello World"}
```

You can also pass data to the invoke command, for example:

![user input](images/userinput.png)
>```sh
> echo -n '{"name":"Bob"}' | fn invoke labapp-NNN pythonfn
>```

```json
{"message":"Hello Bob"}
```

The JSON data was parsed and since `name` was set to "Bob", that value is passed
in the function response.

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
use the following command to see only versions of pythonfn:

![user input](images/userinput.png)
>```sh
> docker images | grep pythonfn
>```

You should see something like:

```sh
phx.ocir.io/mytenancy/myuser/pythonfn   0.0.2    5c0b79de04e8   15 minutes ago  66.3MB
phx.ocir.io/mytenancy/myuser/pythonfn   0.0.1    4936dbed0df6   40 minutes ago  66.3MB
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
pythonfn  phx.ocir.io/mytenancy/myuser/pythonfn:0.0.2    ocid1.fnfunc.oc1.us-phoenix-1.aaaaaaaaacm4u6futn2q34fh4hbkirwsxdffss42kvd3kl6xxakkful5yehq
```

## Wrap Up

Congratulations!  In this lab you've accomplished a lot.  You've created
your first function, deployed it to Oracle Functions and invoked it!

NEXT: [*Unit Testing with Python*](4b-Unit-Testing-python.md), UP: [*Labs*](1-Labs.md), HOME:
[*INDEX*](README.md)
