# Communicating with the UI

> Unfinished

![article1_img7](https://cloud.githubusercontent.com/assets/2152766/11361011/b9d015e2-9281-11e5-91b1-9164dcbde9ca.png)

As mentioned previously, once [pyblish-qml][] is up and running, it will also be set-up to listen on port `9090` so all we need to do in order to speak with it is connect.

> Ensure you have either a host or [pyblish-tray][] already running before attempting this.

```python
import xmlrpclib
proxy = xmlrpclib.ServerProxy("http://127.0.0.1:9090")
proxy.show(9001) # Port number of the a host
```

The port number typically is passed from a host such that the UI knows with whom it is speaking. In this case, as we are testing and getting familiar with things, we pass it in manually.

You can also use the convenience function from within a host provided by the [pyblish-qml][] module.

```python
from pyblish_qml import client
proxy = client.proxy()
proxy.show(9001)
```

The convenience function doesn't need a port number, as it will use the one stored in the environment variable `CLIENT_PORT`, which represents the port number at which the running host is listening.

Here are some of the functionality exposed via the proxy.

- show(int, dict)
- hide()
- quit()
- kill()
- heartbeat(int)
- find_available_port()

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
