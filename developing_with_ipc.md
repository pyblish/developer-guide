# Developing with IPC

The workflow for implementing a new feature into the user interface generally goes like this.

1. Expose a feature of Pyblish through [pyblish-rpc][]
2. Make use of the exposed feature through [pyblish-qml][]

In this article we will have a look at how to do just that.

### Exposing a feature for RPC

As we saw in [A primer on IPC][

[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
[pyblish-qml]: https://github.com/pyblish/pyblish-qml