# UI Communicating with the Host

> Unfinished

![article1_img7](https://cloud.githubusercontent.com/assets/2152766/11360996/972efe54-9281-11e5-90c7-3f4455a2d8b8.png)

Conversely, the UI creates a proxy much like the above and requests data of its own.

Remember there are two processes running, one local and one remote. The remote in the case of the UI is the host. Now let's have a look at what happens on the local side during the processing of a single plug-in from the user interface.

1. User hits publish
2. UI elements and parsed into proxies
3. Proxies are sent to [logic.process][]
4. [logic.process][] delegates the task to the remote

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