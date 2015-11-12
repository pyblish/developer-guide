# 4.3 Communicating with the UI

As mentioned previously, the UI is already set-up to listen on port `9090` so all we need to do in order to speak with it is connect.

```python
import xmlrpclib
proxy = xmlrpclib.ServerProxy("http://127.0.0.1:9090")
proxy.show()
```

You can also use the convenience function provided by the [pyblish-qml][] module.

> Ensure you have [pyblish-qml][] already running before attempting this.

```python
from pyblish_qml import client
proxy = client.proxy()
proxy.show()
```

The convenience function does a few more things, like including a reasonable timeout along with not requiring the user to specify an exact address and port number.

Here are some of the functionality exposed via the proxy.

- show(int, dict)
- hide()
- quit()
- kill()
- heartbeat(int)
- find_available_port()

**Having problems?**

Without the UI running, you will get the familiar error message.

```bash
No connection could be made because the target machine actively refused it
```
