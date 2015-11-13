# A primer on IPC

Developing for the user interface of Pyblish requires an understanding of inter-process communication (IPC), so the following is a quick-start for those unfamiliar with it.

[pyblish-qml][] runs as an individual process on the local machine and communicates with a host via Remote Procedure Calls (RPC) over standard TCP, encapsulated into the [pyblish-rpc][] module.

![image](https://cloud.githubusercontent.com/assets/2152766/11087234/962c0cda-8850-11e5-87d8-db8aacf67df6.png)

The UI generally makes requests to the host, and the host replies with data. 
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

[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc

[01]: http://zguide.zeromq.org/py:all
[02]: http://shop.oreilly.com/product/9780596805838.do
[03]: http://www.amazon.co.uk/Enterprise-Application-Architecture-Addison-Wesley-Signature/dp/0321127420/ref=pd_bxgy_14_img_2?ie=UTF8&refRID=1C160CEZ0ZPX56ZMXYH4
[04]: http://www.amazon.co.uk/Enterprise-Integration-Patterns-Designing-Addison-Wesley/dp/0321200683
[05]: http://www.amazon.co.uk/Service-Design-Patterns-Fundamental-Addison-Wesley/dp/032154420X/ref=asap_bc?ie=UTF8