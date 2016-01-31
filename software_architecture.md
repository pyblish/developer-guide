
# Software Architecture

Now that we know more about the concepts in Pyblish, let's take a closer look at it fits together in technical terms.

### Overview

From afar, Pyblish consists of three major components.

![image](https://cloud.githubusercontent.com/assets/2152766/11087036/b66da078-884e-11e5-8568-f6a5e54382ac.png)

The core is designed and developed in isolation and runs without supporting libraries or dependencies. The remainder is optional but useful, such as a graphical user interface along with the glue involved in bridging a host, such as Autodesk Maya, to Pyblish.

The integration consists of a server and a series of plug-ins, both of which we will talk more about later.

### Module structure

To the user, Pyblish is an executable they run to publish things. To you, there are many decoupled modules involved in making Pyblish work and in making it maintainable long term.

Each module is versioned using [Semantic Versioning][semver] and developed independently. Each hosting source code, issue tracker and wiki.

[semver]: http://semver.org/

Here are some examples of modules and packages.

|                | Module                 | Description
|:---------------|:-----------------------|:-----------
| ![][package]   | [pyblish-win][]        | Officially supported modules and binaries for Windows (package)
| ![][package]   | [pyblish-linux][]      | Officially supported modules and binaries for Linux (package)
| ![][package]   | [pyblish-osx][]        | Officially supported modules and binaries for Linux (package)
| ![][package]   | [pyblish][]          | Officially supported modules (package)
| ![][module]    | [pyblish-base][]            | Base module, the heart of Pyblish.
| ![][module]    | [pyblish-qml][]        | User interface module, the face of Pyblish.
| ![][module]    | [pyblish-tray][]       | The Pyblish control panel
| ![][module]    | [pyblish-rpc][]        | Communication bridge between the core and user interface of Pyblish.
| ![][module]    | [pyblish-integration][]| Supporting module for [pyblish-rpc][]
| ![][module]    | [pyblish-maya][]       | Integration module for Autodesk Maya
| ![][module]    | [pyblish-houdini][]    | Integration module for SideFx Houdini
| ![][module]    | [pyblish-ci][]         | Continuous integration server for Pyblish.
| ![][module]    | [pyblish-event][]      | Reference implementation of a cloud-based event monitor.

### Source structure

The following is the file structure of the core Pyblish module.


|                | Module             | Description
|:---------------|:-------------------|:-----------
| ![][folder]   | [plugins][]         | Default plug-ins.
| ![][folder]   | [vendor][]          | Third party dependencies (no external dependencies).
| ![][folder]   | [\_\_init__.py][] | Global, private variables.
| ![][folder]   | [\_\_main__.py][] | Making this package executable.
| ![][file]     | [api.py][]          | The developer facing interface to Pyblish.
| ![][file]     | [cli.py][]          | Command-line interface, written with the Click support library.
| ![][file]     | [compat.py][]       | Backwards and forward-compatibility features.
| ![][file]     | [lib.py][]          | Helper functions used across surrounding Python modules.
| ![][file]     | [util.py][]         | Convenience module for publishing via scripting.
| ![][file]     | [version.py][]      | The semantic version of this Pyblish module.


And finally, the most relevant files in terms of developing for Pyblish:


|               | Module              | Description
|:--------------|:--------------------|---------------------
| ![][file]     | [plugin.py][]       | Plug-in definition, registration and discovery on disk.
| ![][file]     | [logic.py][]        | The brains of processing.

### Processing Pipeline

As soon as the user hits "publish", three things happen.

1. Create
2. Sort
3. Run

Or, more specifically.

1. A new [Context][] is instantiated and plug-ins discovered.
2. [logic.process][] determines which plug-in to run next.
3. [plugin.process][] performs the actual processing.

The [logic][] module is used primarily in the interfaces with which a user interacts, namely the command-line interface and graphical user interface. It only handles delegating of tasks to functions capable of producing results. In this case, a literal dictionary called [results][].

> We'll see a bit later how this works with inter-process communication.

[plugin.process][] then is the brains of the operation and is the function actually running your plug-in, passing along the currently active [Instance][]. Its responsibility is to..

1. Intercept log messages
2. Handle exceptions
3. Format the resulting data
4. Build the [results][] dictionary.

During processing, the context may be extended with [Instance][]'s. These may be added at any part of the pipeline, but only subsequent plug-ins will see them, which is why they are typically added as early as possible in what is commonly referred to as "Collection". By the end of processing, a [results][] dictionary is created and stored within the [Context][].

The [results][] dictionary is appended to a list within the [Context][] - accessible as `data["results"]` - and captures information regarding the order in which plug-ins and instances are processed and all messages therein. It is primarily intended for graphical user interfaces to visualise the events that occur during processing, but is open to developers to produce visualisations of their own. For example you may be interested in storing the results in a log somewhere on the cloud for auditing purposes such that one may "go back in time" and inspect what actually went on back then.

<div class="modified-date">{{ file.mtime }}</div>

[file]: https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png
[folder]: https://cloud.githubusercontent.com/assets/2152766/11087071/f1c6172c-884e-11e5-87b2-d2f502a01961.png
[package]: https://cloud.githubusercontent.com/assets/2152766/11087037/bd4964ea-884e-11e5-928a-3e3c84f37662.png
[module]: https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png

[plugins]: https://github.com/pyblish/pyblish/tree/master/pyblish/plugins
[vendor]: https://github.com/pyblish/pyblish/tree/master/pyblish/vendor
[\_\_init__.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/__init__.py
[\_\_main__.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/__main__.py
[api.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/api.py
[cli.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/cli.py
[compat.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/compat.py
[lib.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/lib.py
[util.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/util.py
[version.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/version.py
[plugin.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/plugin.py
[logic.py]: https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py

[pyblish-base]: https://github.com/pyblish/pyblish-base
[pyblish-maya]: https://github.com/pyblish/pyblish-maya
[pyblish-houdini]: https://github.com/pyblish/pyblish-houdini
[pyblish-nuke]: https://github.com/pyblish/pyblish-nuke
[pyblish-hiero]: https://github.com/pyblish/pyblish-hiero
[pyblish-magenta]: https://github.com/pyblish/pyblish-magenta
[pyblish-napoleon]: https://github.com/pyblish/pyblish-napoleon
[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
[pyblish-win]: https://github.com/pyblish/pyblish-win
[pyblish-linux]: https://github.com/pyblish/pyblish-linux
[pyblish-osx]: https://github.com/pyblish/pyblish-osx
[pyblish]: https://github.com/pyblish/pyblish
[pyblish-tray]: https://github.com/pyblish/pyblish-tray
[pyblish-integration]: https://github.com/pyblish/pyblish-integration
[pyblish-ci]: https://github.com/pyblish/pyblish-ci
[pyblish-event]: https://github.com/pyblish/pyblish-event

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
