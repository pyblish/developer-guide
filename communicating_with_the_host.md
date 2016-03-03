# UI Communicating with the Host

![article1_img7](https://cloud.githubusercontent.com/assets/2152766/11360996/972efe54-9281-11e5-90c7-3f4455a2d8b8.png)

Conversely, the UI creates a proxy much like the host does.

Remember there are two processes running, one local and one remote. The remote in the case of the UI is the host. Now let's have a look at what happens on the local side during the processing of a single plug-in from the user interface.

1. User hits publish
2. UI elements and parsed into proxies
3. Proxies are sent via [pyblish-rpc.client][] from the GUI
4. Proxies are recieved via [pyblish-rpc.service][] on the remote

[pyblish-rpc.client]: https://github.com/pyblish/pyblish-rpc/blob/master/pyblish_rpc/client.py
[pyblish-rpc.service]:https://github.com/pyblish/pyblish-rpc/blob/master/pyblish_rpc/service.py

<br>

### Example

In Maya, once `pyblish_maya.setup()` has been run, this is how the GUI connects with it.

```python
import xmlrpclib
proxy = xmlrpclib.ServerProxy("http://127.0.0.1:9001/pyblish")
proxy.context()
# {'data': {'cwd': 'C:\\Program Files\\Autodesk\\Maya2015', 'pyblishRPCVersion': '0.2.0', 'currentFile': '.', 'results': 'Not supported', 'connectTime': '2016-03-03T06:32:26.449000Z', 'pythonVersion': '2.7.3 (default, May  8 2013, 09:43:48) [MSC v.1700 64 bit (AMD64)]', 'date': '2016-03-03T06:27:08.405000Z', 'host': 'maya, mayapy, mayabatch, python', 'pyblishServerVersion': '1.3.1', 'user': 'marcus', 'workspaceDir': 'C:\\Users\\marcus\\Documents\\maya\\projects\\default', 'current_file': '.', 'workspace_dir': 'C:\\Users\\marcus\\Documents\\maya\\projects\\default', 'port': 9001}, 'children': []}
```

Other integrations exhibit similar interfaces.

<br>

Here are some of the functionality exposed via the proxy.

| Command          | Description
|------------------|:--------------------
| `test()`         | Return currently registered test
| `ping()`         | Used to check connectivity
| `stats()`        | Return statistics about the API
| `reset()`        | Run collection
| `context(int)`   | Return current context
| `discover()`     | Return discovered plug-ins
| `emit()`         | Remotely emit an event

<br>

### Serialisation

Because Python in one process does not have access to the local variables of another process, the delegation involves a conversion of the local variables and instantiated classes into something we can transfer without loss of information and also use in order to recreate the exact scenario on the remote; JSON.

The conversion looks something like this.

```python
class format_instance(instance):
  """Capture the essence of an Instance, just enough to visualise in a UI"""
  return {
      "name": instance.name,
      "id": instance.id,
      "children": children,
      "data": format_data(instance.data)
  }
```

This is then the data used in the UI, in place of the original instance on the remote. Locally, the data is wrapped up in a class called `InstanceProxy` which looks something like this.

```python
class InstanceProxy(pyblish.api.Instance):
    @classmethod
    def from_json(cls, instance):
        self = cls(instance.pop("name"])
        copy = instance
        copy["data"] = copy.pop("data")
        self.__dict__.update(copy)
        self[:] = instance["children"]
        return self

    def to_json(self):
        return {
            "name": self.name,
            "id": self.id,
            "data": self.data,
            "children": list(self),
        }
```

<div class="modified-date">{{ file.mtime }}</div>

[logic.process]: https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py
[plugin.process]: https://github.com/pyblish/pyblish/blob/master/pyblish/plugin.py