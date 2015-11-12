
### 4. User Interface

The current user interface started as a reference implementation with which other user interfaces were to be developed. Since its introduction, the goal of a user interface for publishing has been made more clear and in response a series of improvements and features was added to this implementation.

Due to this origin however, the foundation of its current incarnation is far from strong. Alternatives are most welcome, especially from developers with experience in UI/UX and optionally Qt and Python.

##### 4.1. Why QML?

TLDR; In order to remain competitive.

[pyblish-qml][] is the officially supported user interface for Pyblish and is written in Python 2.7 and QML 2 with PyQt5.

QML is a declarative language developed by The Qt Company to develop OpenGL accelerated graphical user interfaces. QML was originally inspired by CSS and aims to simplify the development of graphical user interfaces by decoupling code written for visuals from code written for business logic, enabling artists to step in with less of a programming background, and programmers to worry less about cosmetics.

QML was chosen for being the successor to the traditional Widgets, for its novelty and speed in terms of how software is developed but most importantly in order to, as an individual/small company, be able to compete with big organisations who may share my goals but who are stuck writing widgets for the foreseeable future.

All roads point to PySide and Qt 4 being the most lucrative development framework for film and games, but if one is to inspire change and take any kind of lead, one must "think different".

Many integration endpoints, such as Autodesk Maya, facilitate integration by providing the developer with an ability to *mix* custom software with the host. For example, one can both augment and append to existing widgets within the host itself.

This has many benefits.

- Custom software can get by with zero dependencies, apart from the source code. This makes deployment effortless.
- Communication between host and custom software is equal compared with what already happening internally.
- Custom software will be more familiar to the end-user who may in turn wish to extend it further.

However it doesn't come without cons.

- Not all software provides this level of integration, splitting custom software into two parts.
- Each provider provides it differently, meaning bugs appearing in one host may not appear elsewhere, further dividing the codebase.
- Custom software shares the weaknesses of its provider.
  - This includes things such as having threads locked at the whim of the provider.
  - Not facilitating asynchronism, leading to a jittery end-user experience.
  - Not being able to introspect the custom software environment; the provider is in charge and it is proprietary.
- Finally, custom software being limited to past trends and unable to benefit from new developments in the field, impairing your ability to compete.

For hack-and-slash software, written in the heat of production, the provided integration is indispensable. Enabling the rapid development of tools otherwise not possible. For software meant to last, the future is now. Literally. In that one must consider how to ensure survival an ever-changing environment.

##### 4.2. A primer in inter-process communication

Developing for the user interface of Pyblish requires an understanding of inter-process communication (IPC), so the following is a quick-start for those unfamiliar with it.

[pyblish-qml][] runs as an individual process on the local machine and communicates with a host via Remote Procedure Calls (RPC) over standard TCP, encapsulated into the [pyblish-rpc][] module.

![image](https://cloud.githubusercontent.com/assets/2152766/11087234/962c0cda-8850-11e5-87d8-db8aacf67df6.png)

Any host interested in providing their user with a user interface to Pyblish must first:

1. Start listening at a given port number
2. Send this number to [pyblish-qml][] - the client

The client has a server of its own, listening for incoming requests in the same manner as the host. The address of this server is fixed at `9090`.

![image](https://cloud.githubusercontent.com/assets/2152766/11087259/c115d37c-8850-11e5-8a0f-c7966516cf8d.png)

Both servers uses the Python Standard Library XML-RPC module which works like this.

```python
from SimpleXMLRPCServer import SimpleXMLRPCServer as Server

# Implement functionality
def function(a, b):
  return a + b

# Expose functionality through server
ip, port = "127.0.0.1", 10000
server = Server((ip, port))
server.register_function(function)
server.serve_forever()
```

This will start an infinite loop, listening for calls to `function(a, b)`, with which we can communicate from a separate process like this.

```python
import xmlrpclib
proxy = xmlrpclib.ServerProxy("http://127.0.0.1:10000")
print proxy.function(5, 10)
# 15
```

And that's all there is to it.

> Take a break. Have a Kit-kat

Take a moment to really absorb the above information, experiment with it, try to break it. This technique lays the foundation not only for how [pyblish-qml][] communicates with the core, but for how inter-process communication works everywhere. There are many variations to this approach that you should explore to really get a sense of the possibilities of this technique.

Here are some well-known variations for you to Google about.

- Remote Procedure Calls
- Message Queues
- RESTful

And here is some recommended reading of particular interest.

- [The ZeroMQ Guide][01]
- [RESTful in Practice][02]
- [Enterprise Application Integration][03]
- [Enterprise Integration Patterns][04]
- [Service Design Patterns][05]

> Break over

##### 4.3 Communicating with the UI

As mentioned above, the UI is already set-up to listen on port `9090` so all we need to do in order to speak with it is connect.

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

<details>
<summary>Having problems?</summary>

Without the UI running, you will get the familiar error message.

```bash
No connection could be made because the target machine actively refused it
```
</details>

Here are some of the functionality exposed via the proxy.

- show(int, dict)
- hide()
- quit()
- kill()
- heartbeat(int)
- find_available_port()

##### 4.4. UI Communicating with the Host

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

[maya]: https://github.com/pyblish/pyblish-maya
[houdini]: https://github.com/pyblish/pyblish-houdini
[nuke]: https://github.com/pyblish/pyblish-nuke
[hiero]: https://github.com/pyblish/pyblish-hiero
[magenta]: https://github.com/pyblish/pyblish-magenta
[napoleon]: https://github.com/pyblish/pyblish-napoleon
[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc

[Context]: https://github.com/pyblish/pyblish.api/wiki/Context
[Instance]: https://github.com/pyblish/pyblish.api/wiki/Instance
[results]: https://github.com/pyblish/pyblish.api/wiki/results
[logic]: https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py
[logic.process]: https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py
[plugin.process]: https://github.com/pyblish/pyblish/blob/master/pyblish/plugin.py

[1]: https://github.com/pyblish
[2]: https://github.com/pyblish
[3]: https://github.com/pyblish/pyblish-integration
[4]: https://github.com/pyblish/pyblish/blob/master/pyblish/__init__.py

[01]: http://zguide.zeromq.org/py:all
[02]: http://shop.oreilly.com/product/9780596805838.do
[03]: http://www.amazon.co.uk/Enterprise-Application-Architecture-Addison-Wesley-Signature/dp/0321127420/ref=pd_bxgy_14_img_2?ie=UTF8&refRID=1C160CEZ0ZPX56ZMXYH4
[04]: http://www.amazon.co.uk/Enterprise-Integration-Patterns-Designing-Addison-Wesley/dp/0321200683
[05]: http://www.amazon.co.uk/Service-Design-Patterns-Fundamental-Addison-Wesley/dp/032154420X/ref=asap_bc?ie=UTF8