---
title: Performing custom actions on uncaught python exceptions
date: 2024-01-06
public: true
---

# Performing custom actions on uncaught python exceptions

I wanted to make something that sent a request to a webserver whenenever my
python application crashed, which would be logged in a database. It's possible
to just wrap the `main()` function in a `try: except:` block, but that's ugly
and after some quick googling I realised there's a nicer way to do this.

Python's `sys.excepthook` runs whenever there's an uncaught exception, and you
can just write your own function to override this.


Here's an example sending the traceback to a server with a http post request,
before raising the usual exception.



```python
# exception_handling.py

import sys
import traceback
from types import TracebackType
from typing import Type

import requests


def set_exception_endpoint(app_name: str, url: str) -> None:

    def handle_exception(
        exc_type: Type[BaseException],
        exc_value: BaseException,
        exc_traceback: Optional[TracebackType],
    ) -> None:
        """over-ride sys.excepthook with a requests.post"""
        if issubclass(exc_type, KeyboardInterrupt):
            # probably don't want to be recording an KeyboardInterrupts
            sys.__excepthook__(exc_type, exc_value, exc_traceback)
            return
        traceback_str = "".join(
            traceback.format_exception(exc_type, exc_value, exc_traceback)
        )
        data = {
            "app": app,
            "traceback": traceback_str,
        }
        resp = requests.post(url, json=data, timeout=0.5)
        sys.__excepthook__(exc_type, exc_value, exc_traceback)

    sys.excepthook = handle_exception
```

This can then be imported and initialised at the top of an application:

```python
from exception_handling import set_exception_endpoint


set_exception_endpoint("my_crashing_app", "http://localhost:5000")


def main():
    # stuff goes here
    pass


if __name__ == "__main__":
    main():
```

I'm trying this at work where many of my applications run in docker containers
on VMs or k8 pods where I don't have much visibility to logs or any crash reports.
