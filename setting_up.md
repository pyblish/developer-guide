# Setting up on Windows

In order to get started developing for Pyblish on the Windows platform, let's set-up your environment.

> Much of this also applies to other platforms.

You will need:

1. Git
1. Python (2.7 x64)
1. Pyblish
 
If you are interested in working with the UI, then you will also need PyQt5.

### Setting up PyQt

Assuming you already have Git and Python installed on your local machine, let's start with PyQt.

```bash
# Open up a terminal and run this.
mkdir pyblishdev
cd pyblishdev
git clone https://github.com/pyqt/python-qt5.git
set PYTHONPATH=%CD%\python-qt5;%PYTHONPATH%
python -c "import util;util.createqtconf()"
```

> The last command will attempt to physically create a file called `qt.conf` local to your installation of Python.

Now let's test the installation to make sure all is well.

```python
# Open up Python and run this.
import sys
from PyQt5 import QtWidgets
app = QtWidgets.QApplication(sys.argv)
button = QtWidgets.QPushButton("Hello")
button.setFixedSize(400, 400)
button.show()
app.exec_()
```

### Setting up Pyblish

To develop for Pyblish means to fork relevant repositories onto your GitHub account, make a change and finally submit a pull-request.

The exact repositories to fork depends on which areas you are interested in developing for. See the list below for hints and fork the ones of interest.

Replace the addresses to the ones below with your forked equivalent and clone.


```bash
:: Required
git clone https://github.com/pyblish/pyblish

:: For UI development
git clone https://github.com/pyblish/pyblish-qml
git clone https://github.com/pyblish/pyblish-rpc

:: For any form of integration or integration with a new host
git clone https://github.com/pyblish/pyblish-integration

:: For integration with an existing host
git clone https://github.com/pyblish/pyblish-maya
git clone https://github.com/pyblish/pyblish-nuke
git clone https://github.com/pyblish/pyblish-houdini
git clone https://github.com/pyblish/pyblish-hiero
git clone https://github.com/pyblish/pyblish-modo

:: To be ready for anything
:: COPY PASTE ALL OF THE ABOVE
```

### Repository Layout

Each of these repositories are set-up in the same way; at the root level there is a Python package. So what we need to do is somehow expose this package onto your PYTHONPATH.

How you choose to do that is not important, but here are some ideas.

1. Append the absolute path to each repository to your PYTHONPATH
2. Create a "junction" to each Python package within each repository in some directory on your PYTHONPATH.

Here is an example of how to append the most common modules.

```bash
cd pyblishdev
set PYTHONPATH=%CD%/pyblish;%CD%/pyblish-rpc;%CD%/pyblish-qml
```

If you are working with [pyblish-qml][] also make sure to run this test.

```bash
python -m pyblish_qml.compat
# Qt detected..
# ==============================================================================
# 
#  - Success!
# 
# c:\python27\python.exe is well suited to run Pyblish QML
# 
# ==============================================================================
```

### Testing out the installation

In a new terminal with the PYTHONPATH set, let's run the test and ensure all is well.

```bash
cd pythondev\pyblish
python run_testsuite.py
```

<div class="modified-date">{{ file.mtime }}</div>

[pyblish-qml]: https://github.com/pyblish/pyblish-qml