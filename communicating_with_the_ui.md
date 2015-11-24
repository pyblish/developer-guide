# Communicating with the UI

> Unfinished

![article1_img7](https://cloud.githubusercontent.com/assets/2152766/11361011/b9d015e2-9281-11e5-91b1-9164dcbde9ca.png)

As mentioned previously, once [pyblish-qml][] is up and running, it will also be set-up to listen on port `9090` so all we need to do in order to speak with it is connect.

> Ensure you have either [pyblish-qml][] running such as in debug-mode, via a host or [pyblish-tray][] before attempting this.

```python
import xmlrpclib
proxy = xmlrpclib.ServerProxy("http://127.0.0.1:9090")
proxy.show(9001) # Port number of the a host
```

The `9001` is your "business card". It let's [pyblish-qml][] know that the client looking to communicate with it can be communicated with in return via this port. In this way, communicating is bi-directional.

Under normal circumstances, this port number is established during host initialisation.

```python
import pyblish_maya
pyblish_maya.setup()  # A port number is assigned
```

The port number is then stored "behind the scenes" in an environment variable called `PYBLISH_CLIENT_PORT` and it is this number which is automatically passed upon the user pressing the **Publish** menu-item.

```python
import os
assert os.environ["PYBLISH_CLIENT_PORT"] == "9001"
```

> Note that you may see a different number, such as 9002, depending on how many hosts you currently have open. Each host has a unique number, starting from 9001.

You can also use the convenience function from within a host provided by the [pyblish-qml][] module.

```python
from pyblish_qml import client
proxy = client.proxy()
proxy.show(9001)
```

The convenience function is much the same, but also includes a reasonable time-out for when the host is unresponsive.

Here are some of the functionality exposed via the proxy.

| Command                     | Description
|-----------------------------|:--------------------
| `show(int, dict)`           | Connect with pyblish-qml, takes port and window-settings.
| `hide()`                    | Hide the currently open pyblish-qml window, without quitting.
| `quit()`                    | Quit the process, including clean-up.
| `kill()`                    | Forcefully quit the process.
| `heartbeat(int)`            | Tell [pyblish-qml][] that you are alive, takes a port number.
| `find_available_port()`     | Query [pyblish-qml][] for the next available, unique port number.

<br>

**Having problems?**

Without the UI running, you will get the familiar error message.

```bash
No connection could be made because the target machine actively refused it
```

<div class="modified-date">{{ file.mtime }}</div>

[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-tray]: https://github.com/pyblish/pyblish-tray
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
