# Developing with IPC

The workflow for implementing a new feature into the user interface generally goes like this.

![image](https://cloud.githubusercontent.com/assets/2152766/11148000/ded2265a-8a11-11e5-9fd5-60645642c2cf.png)

1. Expose a feature of Pyblish through [pyblish-rpc][]
2. Make use of the exposed feature through [pyblish-qml][]

In this article we will have a look at how to do just that.

### Overview

As we saw in [A primer on IPC](interprocess-communication.md), to be able to use functionality in one process from another process, we *register* a function with the RPC server.

Aside from registering a plain function, we are also able to register an instance of a class, called "service". We use it in place of freestanding functions in order to share memory state amongst one or more calls, such as maintaining reference to discovered plug-ins and instances.

The default service is located in [pyblish-rpc/service.py][] and looks like this.

```python
class RpcService(object):
    def __init__(self):
        self._context = None
        self._plugins = None
        self._provider = None

        self.reset()

    def ping(self):
        """Used to check connectivity"""
        return {
            "message": "Hello, whomever you are"
        }
```

This service is registered by default when launching any integrated host. If you have Pyblish installed, try launching any host and type this in.

```python
import xmlrpclib
proxy = xmlrpclib.ServerProxy("http://127.0.0.1:9001/pyblish")
proxy.ping()
# {"message": "Hello, whomever you are"}
```

> Note the additional path `/pyblish`.

You can find the exposed port number of your host via the environment variable `QML_CLIENT_PORT` which in this case is `9001`. If the variable isn't available, make sure the Pyblish has been installed correctly and that the integration is loaded.

- [Installation guide](https://github.com/pyblish/pyblish/wiki#installation)

### Exposing a feature for RPC

Now that we know how features are exposed, let's try exposing something we can use with [pyblish-qml][].

```python
class RpcService(object):
    def add_two_numbers(self, a, b):
        return a + b
```

With the method added, let's have a look at how we can use this in [pyblish-qml][].

### Using a feature

Before diving into the QML side, let's stick to a standalone Python interpreter.

QML instantiates `Proxy` from [pyblish-rpc/client.py][] and uses it for any and all communication with Pyblish within a host. Let's try doing that in a plain Python interpreter.

```python
from pyblish_rpc.client import Proxy
proxy = Proxy(9001)
```

The port is normally passed to QML from the host during the point at which a user triggers its visibility; e.g. by pressing "Publish" from the file-menu or by calling `.show()` from it's integration, such as `pyblish_maya.show()`.

For this example, we will assume a host is running at `9001`.

Now we can call any function exposed via this service.

```python
# Retrieve a list of plugins
plugins = proxy.discover()

# Retrieve the current context
context = proxy.context()
```

Including the one we just implemented.

```python
print(proxy.add_two_numbers(1, 2))
# 3
```

### Using a feature in the controller

In a typical model/view/controller application, the controller is the part where logic and event handling occurs. This is where we will make use of our feature.

Open up [pyblish-qml/control.py][] and add the calling method to the `Controller` class.

```python
# pyblish-qml/pyblish_qml/control.py
class Controller(QtCore.QObject):
    ...
    def add_two_numbers(self, a, b):
        result = self.host.add_two_numbers(a, b)
        print("The result is: %s" % result)
        return result
    ...
```

`self.host` represents the currently active host and is an instance of the `Proxy` class we saw above.

Whenever a host connects/shows the GUI, `Proxy` is reinstantiated to reflect the change.

### Using a feature in QML

To call upon a feature from QML directly, all we need to do is decorate the method we just made.

```python
# pyblish-qml/pyblish_qml/control.py
class Controller(QtCore.QObject):
    ...
    @QtCore.pyqtSlot(int, int, result=int)
    def add_two_numbers(self, a, b):
        result = self.host.add_two_numbers(a, b)
        print("The result is: %s" % result)
        return result
    ...
```

We are then able to call this function from within QML.

```js
// pyblish-qml/pyblish_qml/qml/Overview.qml
Footer {
    id: footer

    mode: overview.state == "publishing" ? 1 : overview.state == "finished" ? 2 : 0

    width: parent.width
    anchors.bottom: parent.bottom

    onReset: {
        // Run our function at the touch of the reset-button
        app.add_two_numbers(2, 4)
        app.reset()
    }
}
```

Notice how the controller is exposed as a global variable in QML called `app`. Try launching the GUI and pressing the Reset button, and keep an eye out in the output.

```bash
The result is: 6
```

[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-qml/control.py]: https://github.com/pyblish/pyblish-qml/blob/master/pyblish_qml/control.py
[pyblish-rpc/server.py]: https://github.com/pyblish/pyblish-rpc/blob/master/pyblish_rpc/server.py
[pyblish-rpc/client.py]: https://github.com/pyblish/pyblish-rpc/blob/master/pyblish_rpc/client.py
[pyblish-rpc/service.py]: https://github.com/pyblish/pyblish-rpc/blob/master/pyblish_rpc/service.py