# Getting started with Pyblish QML

Running Pyblish QML as a user involves hitting the "Publish" menu-item of your host, or running something like this.

```python
# In a Python interpreter
import pyblish_maya
pyblish_maya.show()
```

As a developer however, we need more control. And output.

So instead, it is recommended that you run the application directly.

```python
python -m pyblish_qml --debug
```

> This will launch QML in "debug mode". 

Practically, you would run this from your text editor, such that you can edit it and see the changes directly.

**See also**

- [app.py][1]

<br>
<br>
<br>

### Troubleshooting

If you run into trouble running the above, ensure you have the dependencies on your `PATH` and `PYTHONPATH`, otherwise skip to [Mocking](#mocking) below.

- [pyblish][]
- [pyblish-qml][]
- [pyblish-rpc][]

As an additional test, to test for example the bit-ness of your Python interpreter, make sure this passes.

```python
python compat.py
```

<br>
<br>
<br>

### Mocking

Once up, you'll notice a bunch of plug-ins and instances.

![image](https://cloud.githubusercontent.com/assets/2152766/11264567/88bd37da-8e8c-11e5-9a27-0efbc9b3a5c0.png)

Each plug-in is coming from [pyblish-rpc][] as expected, but are mocks of real plug-ins to help you develop features and fix bugs without having to produce real content from which to produce instances or real plug-ins from which to see whether a particular feature or bug has been resolved.

The plug-ins you see excersize 100% of Pyblish's capabilities and it's UI.

You can browse, add or remove any of these via [pyblish-rpc/mocking.py][], it's a good habit to do this whenever you develop anything you would like to see in action and make sure it works.

<br>
<br>
<br>

### Adding your own mock

Let's add one now.

1. Open [pyblish-rpc/mocking.py][]
2. Create a plug-in.

   ```python
class MyNewPlugin(pyblish.api.Plugin):
       def process(self):
           self.log.info("Ran MyNewPlugin")
```
3. Add it to to the `plugins` list at the bottom.
4. Restart the UI

You should now see your plug-in in the list of available plug-ins; in this case, at the top, as it was a subclass of [Plugin][].

Running the publish will excersize all available plug-ins, including yours, and if the result and output is as expected, you win!

![image](https://cloud.githubusercontent.com/assets/2152766/11265013/991ffa46-8e90-11e5-9ff5-d0a9be324e0e.png)

<br>
<br>
<br>

### Gotchas

When running the application in this way, technically you are running both a client and the server. In the next section, we will talk more about how Pyblish QML runs in terms of inter-process communication, but keep in mind that you might encounter some of these issues as you develop.

> Q: I added to mocking.py, but it isn't showing up!

A: Try editing mocking.py in such a way that it is an invalid Python source file, such as by referencing a non-existant variable, or just by removing everything inside of it, and run the UI again.

Does that work?

If it did, there's a chance you have a server running in the background that isn't being shutdown when closing the GUI, hence not bothering to read your changes to the file.

Have a look in your process explorer for a Python process and kill it.


<br>
<br>
<br>

### Custom Debug Mode

The above debug mode will automatically include a number of plug-ins to test every possible feature of Pyblish QML. It's what we use to visually confirm that the program is running as expected.

It also prevents you from registering your own plug-ins, which may hinder development.

The following snippet is an example of how you can setup your own debug mode, with a locally running "host" that you can register your own plug-ins with.

**Usage**

```bash
$ python qml_custom_debug.py
```

**qml_custom_debug.py**

```python
from pyblish import api
from pyblish_rpc import server, client
from pyblish_qml.app import Application, APP_PATH


def launch():
    """Launch Pyblish QML with mock host"""

    app = Application(APP_PATH)
    app.listen()
    app.__debugging__ = True

    # This is to port at which we'll setup our mock host
    host_port = 6000
    proxy = client.Proxy(host_port)

    # Is there already a host here?
    if not proxy.ping():
        server.start_async_production_server(host_port)
        print("Running mocked RPC server @ 127.0.0.1:%s" % host_port)

    app.show_signal.emit(host_port, {
        "ContextLabel": "My Context",
        "WindowTitle": "My Pyblish",
        "WindowSize": (430, 600)
    })

    print("Pyblish QML running @ 127.0.0.1:9090")

    return app.exec_()


if __name__ == '__main__':
    # Because the host is in the same process as the GUI,
    # it's safe to declare and register plug-ins here.

    class MyPlugin(api.ContextPlugin):
        order = api.CollectorOrder

        def process(self, context):
            self.log.info("Hello World")

    api.register_plugin(MyPlugin)

    launch()
```

<div class="modified-date">{{ file.mtime }}</div>

[1]: https://github.com/pyblish/pyblish-qml/blob/master/pyblish_qml/app.py
[Plugin]: https://github.com/pyblish/pyblish.api/wiki/Plugin
[python-qt5]: https://github.com/pyqt/python-qt5
[pyblish]: https://github.com/pyblish/pyblish
[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
[pyblish-rpc/mocking.py]: https://github.com/pyblish/pyblish-rpc/blob/master/pyblish_rpc/mocking.py