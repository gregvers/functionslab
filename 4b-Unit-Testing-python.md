# Unit Testing with Pytest

The [Fn Python FDK (Function Development Kit)](https://github.com/fnproject/fdk-python)
provides you with the developer experience for unit testing
Python functions.

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

Your testing environment requires Python FDK and pytest to be installed.

You can run the tests by running from the function's directory:
```shell
pytest -v -s --tb=long
```
You should see something like:
```shell
============================================================== test session starts ===============================================================
platform darwin -- Python 3.7.4, pytest-4.0.1, py-1.8.0, pluggy-0.13.0 -- /usr/local/opt/python/bin/python3.7
cachedir: .pytest_cache
rootdir: /Users/gverstra/code/fn-tests/pythonfn, inifile:
plugins: asyncio-0.9.0
collected 2 items                                                                                                                                

test_func.py::test_func_without_data 2019-09-20 15:27:09,558 - asyncio - DEBUG - Using selector: KqueueSelector
2019-09-20 15:27:09,559 - asyncio - DEBUG - Using selector: KqueueSelector
'NoneType' object has no attribute 'getvalue'
{'content-type': 'application/json', 'fn-fdk-version': '0.1.7'}
PASSED
test_func.py::test_func_with_data 2019-09-20 15:27:09,561 - asyncio - DEBUG - Using selector: KqueueSelector
{'content-type': 'application/json', 'fn-fdk-version': '0.1.7'}
PASSED

============================================================ 2 passed in 0.10 seconds ============================================================
```

## Wrap Up

Congratulations! You've just completed an introduction to the Fn Python FDK.
There's so much more in the FDK than we can cover in a brief introduction.  Next
we'll take a look at troubleshooting.  We saw a little of this when our tests
failed but there are a few more techniques available.

NEXT: [*Troubleshooting*](5-Troubleshooting.md), UP: [*Labs*](1-Labs.md), HOME:
[*INDEX*](README.md)
