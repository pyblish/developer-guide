# 6. Writing a UI

In this example we're going to write a graphical user interface from scratch. It will be simple, but should be enough to illustrate how [pyblish-qml][] works.

### 6.1 Getting started

Here is the UI we are going to develop.

![](https://gist.github.com/mottosso/1443b177e15710ee36e4/raw/eada987dcc4f05b1fd0627ec74116bc0aadd7a32/demo.gif)

It illustrates a number of things.

1. How to develop a GUI using PySide
2. How to mock a series of plug-ins for development purposes
2. How to list plug-ins from Pyblish
3. How to run collection and list collected instances
4. How to interact with Pyblish, by publishing
5. How to visualise the results of the publish

Granted this example is rather simplistic, and doesn't include things such as.

1. Log output
2. Feedback on which instance and which plug-in is actually running
3. Safety in case the user hits the button too fast

And so on. But it should be enough for us to understand how things works under the hood of [pyblish-qml][]

### 6.2 Source Code

The full source code of this example is included in this Gist.

- https://gist.github.com/mottosso/1443b177e15710ee36e4

Feel free to try it out at home, simply copy-paste the contents of `main.py` into your favourite text editor or Script Editor (Maya, Nuke) and we'll cover what's inside next.

### 6.3 Setup

To start off, we need a QApplication and a window.

```python
import sys
from PySide import QtGui

class Window(QtGui.QWidget):
    def __init__(self, parent=None):
    super(self, Window).__init__(parent)

app = QtGui.QApplication(sys.argv)
window = Window()
window.show()
app.exec_()
```

This will run fine outside of a host, but let's also ensure it runs inside. We can do that using a context manager.

```python
import sys
import contextlib
from PySide import QtGui

class Window(QtGui.QWidget):
    def __init__(self, parent=None):
        super(Window, self).__init__(parent)

@contextlib.contextmanager
def application():
    """Enable running from within and outside host"""
    if not QtGui.qApp:
        app = QtGui.QApplication(sys.argv)
        yield
        app.exec_()
    else:
        yield

with application():
    window = Window()
    window.show()
```

Now our application will run the same regardless of whether we run it within or outside of a host. This can help simplify development a little bit.

### 6.3 Layout

Now let's establish the layout and populate our GUI.

```python
class Window(QtGui.QWidget):
    def __init__(self, parent=None):
        super(Window, self).__init__(parent)
        self.setWindowTitle("Pyblish")
        self.setObjectName("Window")
        self.resize(400, 400)

        header = QtGui.QWidget()
        filler = QtGui.QWidget()

        # Load logo from URL
        pixmap = QtGui.QPixmap()
        pixmap.loadFromData(urllib.urlopen(
            "https://gist.githubusercontent.com/mottosso"
            "/1443b177e15710ee36e4/raw/07e2db113ab0b438c"
            "d586f8b6dcf418491f2c528/pyblish-white.png").read())

        logo = QtGui.QLabel()
        logo.setPixmap(pixmap)

        layout = QtGui.QHBoxLayout(header)
        layout.addWidget(logo)
        layout.addWidget(filler)
        layout.setContentsMargins(0, 0, 0, 0)

        body = QtGui.QWidget()

        instance_list = QtGui.QListWidget()
        plugin_list = QtGui.QListWidget()

        layout = QtGui.QHBoxLayout(body)
        layout.addWidget(instance_list)
        layout.addWidget(plugin_list)
        layout.setContentsMargins(0, 0, 0, 0)

        footer = QtGui.QWidget()
        filler = QtGui.QWidget()
        publish = QtGui.QPushButton(">")
        refresh = QtGui.QPushButton("R")
        publish.setMaximumWidth(30)
        refresh.setMaximumWidth(30)

        layout = QtGui.QHBoxLayout(footer)
        layout.addWidget(filler)
        layout.addWidget(refresh)
        layout.addWidget(publish)
        layout.setContentsMargins(0, 0, 0, 0)

        # Main layout
        layout = QtGui.QVBoxLayout(self)
        layout.addWidget(header)
        layout.addWidget(body)
        layout.addWidget(footer)

        self.instance_list = instance_list
        self.plugin_list = plugin_list
```

### 6.4 Refresh

Now we need to initialise our GUI with contents from Pyblish. We will do this by calling `pyblish.api.discover()` each time the user presses the "Refresh" button.

Here is the function responsible for populating the GUI with results.

```python
def refresh(self):
    self.plugin_list.clear()

    plugins = pyblish.api.discover()
    for plugin in plugins:
        item = QtGui.QListWidgetItem(plugin.__name__)
        self.plugin_list.addItem(item)
```

We'll also need to run collection in order to also display collected instances.

```python
def refresh(self):
    self.plugin_list.clear()
    self.instance_list.clear()

    plugins = pyblish.api.discover()
    for plugin in pyblish.api.discover():
        item = QtGui.QListWidgetItem(plugin.__name__)
        self.plugin_list.addItem(item)

    context = pyblish.util.collect(plugins=plugins)
    for instance in context:
        item = QtGui.QListWidgetItem(instance.data["name"])
        self.instance_list.addItem(item)
```

### 6.5 Mocks

Rather than require an active scene with actual content inside of it, we will mock up a few plug-ins that "fake" a real scene in order to simplify development further.

```python
class CollectFakeInstances(pyblish.api.Collector):
    def process(self, context):
        for name in ("Bob01", "Truck05", "Paperpen11"):
            instance = context.create_instance(name)
            instance.data["fake"] = True
            self.log.info("Adding %s" % name)

class ValidateFakeInstances(pyblish.api.Validator):
    def process(self, instance):
        import random
        self.log.info("Asserting %s randomly" % instance)
        assert random.random() > 0.25

pyblish.api.deregister_all_plugins()
for plugin in (CollectFakeInstances, ValidateFakeInstances):
    pyblish.api.register_plugin(plugin)
```

Now when we run `refresh()`, will will get both plug-ins and instances for us to test with. Win!

###


[pyblish-qml]: https://github.com/pyblish/pyblish-qml