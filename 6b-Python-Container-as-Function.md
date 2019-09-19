# Creating a Function from a Docker Image

This lab walks through how to use a custom Docker image to define an
Fn function.  Although Fn functions are packaged as Docker images, when
developing functions using the Fn CLI developers are not directly exposed
to the underlying Docker platform.  Docker isn't hidden (you can see
Docker build output and image names and tags), but you aren't
required to be very Docker-savvy to develop functions with Fn.

However, sometimes you need to handle advanced use cases and must take
complete control of the creation of the function container image. Fortunately
the design and implementation of Fn enables you to do exactly that.  Let's
build a simple custom function container image to become familiar with the key
elements of the process.

Unlike in previous labs we aren't going to create a function using the
`fn init` command.  We're going to create all the necessary files from scratch.

As you make your way through this tutorial, look out for this icon.
![](images/userinput.png) Whenever you see it, it's time for you to
perform an action.

# Magick Functions

One of the most common reasons for writing a custom Dockerfile for a function
is to install a Linux package that your function needs.  In our example we're
going to use the the ever-popular [ImageMagick](https://www.imagemagick.org) to
do some image processing in our function and while there are several Python libraries for
ImageMagick, they are just wrappers on the underlying native library.  So we'll
have to install the native library in addition to adding the Python library to our
`requirements.txt` dependencies. Let's start by creating the Python function.

## Function Definition

![](images/userinput.png)
> In an **empty folder** create a file named `func.py` and copy/paste the
following as its content:

```python
import io
import json
import tempfile

from fdk import response
from wand.image import Image

def handler(ctx, data: io.BytesIO=None):
    resp = {}
    with tempfile.TemporaryFile() as tempf:
        tempf.write(data.getbuffer())
        tempf.seek(0)
        with Image(file = tempf) as img:
            resp["width"] = img.width
            resp["height"] = img.height

    return response.Response(
        ctx,
        response_data=json.dumps(resp),
        headers={"Content-Type": "application/json"}
    )
```

The function takes a binary image as its argument, writes it to a tmp file, and
then uses ImageMagick to obtain the width and height of the image.

## Declaring Python Dependencies

There are lots of interesting elements to this function but the key one for us
is the use of the "wand" Python library for ImageMagick image processing.  
To use it we need to include it along with our other dependencies in our
`requirements.txt` file.

![](images/userinput.png)
> In same folder as the `func.js` file, create a `requirements.txt` file and
copy/paste the following as its content:

```text
fdk
wand
```

Like with all Python functions, we include the Fn Python FDK as a dependency
along with "wand" for imagemagick image processing.  

## Function Metadata

Now that we have a Python function and it's dependencies captured in the
`requirements.txt` we need a `func.yaml` to capture the function metadata.

![](images/userinput.png)
> In the folder containing the previously created files, create a `func.yaml`
file and copy/paste the following as its content:

```yaml
schema_version: 20180708
name: imagedims
version: 0.0.1
runtime: docker
memory: 256
```

This is a typical `func.yaml` for a Python function except that instead of
declaring the **runtime** as "Python" we've specified "**docker**".  If you were
to type `fn build` right now you'd get the error:

> ```
> Fn: Dockerfile does not exist for 'docker' runtime
> ```

This is because when you set the runtime type to "docker" `fn build` defers to
your Dockerfile to build the function container image--and you haven't defined
one yet!  

One more small point worthy of note is that if you don't declare the `runtime`
property then it will default to `docker` and the CLI will look for a
Dockerfile.

## Default Node.js Function Dockerfile

The Dockerfile that `fn build` would normally automatically generate to build a
Node.js function container image looks like this:

```Dockerfile
FROM fnproject/python:3.6-dev as build-stage
WORKDIR /function
ADD requirements.txt /function/
			RUN pip3 install --target /python/  --no-cache --no-cache-dir -r requirements.txt &&\
			 rm -fr ~/.cache/pip /tmp* requirements.txt func.yaml Dockerfile .venv
ADD . /function/
RUN rm -fr /function/.pip_cache

FROM fnproject/python:3.6
WORKDIR /function
COPY --from=build-stage /function /function
COPY --from=build-stage /python /python
ENV PYTHONPATH=/python
ENTRYPOINT ["/python/bin/fdk", "/function/func.py", "handler"]
```

It's a two stage build with the `fnproject/python:3.6-dev` image containing `pip` and
other build tools, and the `fnproject/python:3.6` image containing just the Python
runtime.  This approach is designed to ensure that deployable function container
images are as small as possible--which is beneficial for a number of reasons
including the time it takes to transfer the image from a Docker respository to
the compute node where the function is to be run.

## Custom Python Function Dockerfile

The `fnproject/python` container image is built on Debian so we'll need to install
the
[ImageMagick Debian package](https://packages.debian.org/buster/imagemagick)
using the `apt-get` package management utility.  You can do this with a Dockerfile
`RUN` command:

```Dockerfile
RUN apt-get update && apt-get install -y imagemagick
```

We want to install ImageMagick into the runtime image, not the build image,
so we need to add the `RUN` command after the `FROM fnproject/python:3.6` command.

![](images/userinput.png)
> In the folder containing the previously created files, create a file named
`Dockerfile` and copy/paste the following as its content:

```Dockerfile
FROM fnproject/python:3.6-dev as build-stage
WORKDIR /function
ADD requirements.txt /function/
RUN pip3 install --target /python/  --no-cache --no-cache-dir -r requirements.txt &&\
		rm -fr ~/.cache/pip /tmp* requirements.txt func.yaml Dockerfile .venv
ADD . /function/
RUN rm -fr /function/.pip_cache

FROM fnproject/python:3.6
RUN apt-get update && apt-get install -y imagemagick
WORKDIR /function
COPY --from=build-stage /function /function
COPY --from=build-stage /python /python
ENV PYTHONPATH=/python
ENTRYPOINT ["/python/bin/fdk", "/function/func.py", "handler"]
```

With this Dockerfile the Python function, its dependencies (including the
"wand" library), and the "imagemagick" Debian package will be
included in an image derived from the base `fnproject/python:3.6` image. We should be
good to go!

## Building and Deploying

Once you have your custom Dockerfile you can simply use `fn build` to build
your function.  Give it a try:

![](images/userinput.png)
>```
> fn -v build
>```

You should see output similar to:

```shell
Building image phx.ocir.io/ocicpm/test/imagedims:0.0.24
FN_REGISTRY:  phx.ocir.io/ocicpm/test
Current Context:  ocicpm
Sending build context to Docker daemon  10.75kB
Step 1/13 : FROM fnproject/python:3.6-dev as build-stage
 ---> aa4e9945b65f
Step 2/13 : WORKDIR /function
 ---> Using cache
 ---> daec60dbb5e6
Step 3/13 : ADD requirements.txt /function/
 ---> Using cache
 ---> 88d7202e929c
Step 4/13 : RUN pip3 install --target /python/  --no-cache --no-cache-dir -r requirements.txt &&		rm -fr ~/.cache/pip /tmp* requirements.txt func.yaml Dockerfile .venv
 ---> Using cache
 ---> 09dfa3e73d0e
Step 5/13 : ADD . /function/
 ---> 92980d582449
Step 6/13 : RUN rm -fr /function/.pip_cache
 ---> Running in 381c9b405e40
Removing intermediate container 381c9b405e40
 ---> 51ae3a8fffa5
Step 7/13 : FROM fnproject/python:3.6
 ---> c31b51a1f190
Step 8/13 : RUN apt-get update && apt-get install -y imagemagick
 ---> Running in 63439818530b
...
Successfully built 0b76f6b5e783
Successfully tagged phx.ocir.io/mytenancy/myuser/imagedims:0.0.24

Function phx.ocir.io/mytenancy/myuser/imagedims:0.0.24 built successfully.
```

Just like with a default build, the output is a container image.  From this
point forward everything is just as it would be for any Fn function. Let's
deploy to your previously created `labapp-NNN` application, where `NNN` is
your lab participant number.


![](images/userinput.png)
>```
> fn deploy --app labapp-NNN
>```

We can confirm the function is correctly defined by getting a list of the
functions in the "tutorial" application:

![](images/userinput.png)
>```
> fn list functions labapp-NNN
>```

**Pro tip**: The fn cli lets you abbreviate most of the keywords so you can
also type `fn ls f labapp-NNN`

You should see output similar to:

```shell
NAME        IMAGE                                                   ID
imagedims   phx.ocir.io/mytenancy/myuser/imagedims:0.0.2  ocid1.fnfunc.oc1.us-phoenix-1.aaaaaaaaacw6cjiagzwc64hhacuj3ssd7c4e37y4kdsdnjbcmduczrcuywfq
```

## Invoking the Function

With the function deployed let's invoke it to make sure it's working as
expected. You'll need a jpeg or png file so either find one on your machine
or download one.  If you've cloned this lab's Git repo you can use the
`3x3.jpg` image that has a height and width of 3 pixels, or you can download
it from the `images` folder in github.

![](images/userinput.png)
>```
> cat 3x3.jpg | fn invoke labapp-NNN imagedims
>```

The first time you invoke the function you'll incur some "cold start" cost. For
this input file you should see the following output:

>```json
>{"width": 3, "height": 3}
>```

# Conclusion

One of the most powerful features of Fn and Oracle Functions is the ability to
use custom-defined Docker container images as functions. This feature makes it
possible to customize your function's runtime environment including letting you
install any Linux libraries or utilities that your function might need. And
thanks to the Fn CLI's support for Dockerfiles it's the same user experience as
when developing any function.

Having completed this lab you've successfully built a function using
a custom Dockerfile. Congratulations!

UP: [*Labs*](1-Labs.md), HOME: [*INDEX*](README.md)
