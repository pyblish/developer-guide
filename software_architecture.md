
# 3. Software Architecture

Now that we know more about the concepts in Pyblish, let's take a closer look at it fits together in technical terms.

### 3.1. Overview

From afar, Pyblish consists of three major components.

![image](https://cloud.githubusercontent.com/assets/2152766/11087036/b66da078-884e-11e5-8568-f6a5e54382ac.png)

The core is designed and developed in isolation and runs without supporting libraries or dependencies. The remainder is optional but useful, such as a graphical user interface along with the glue involved in bridging a host, such as Autodesk Maya, to Pyblish.

The integration consists of a server and a series of plug-ins, both of which we will talk more about later.

### 3.2. Module structure

The the user, Pyblish is an executable they run to publish things. To you, there are many decoupled modules involved in making Pyblish work and in making it maintainable long term.

Each module is versioned using [Semantic Versioning][semver] and developed independently. Each hosting source code, issue tracker and wiki.

[semver]: http://semver.org/

Here are some examples of modules and packages.

<table>
<thead>
<th></th>    <th>Module</th> <th>Package</th>    <th>Description</th>
</thead>
<tbody>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087037/bd4964ea-884e-11e5-928a-3e3c84f37662.png"></td>
<td><a link="https://github.com/pyblish/pyblish-win">pyblish-win</a></td>
<td></td>
<td>Officially supported modules and binaries for Windows (package)</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087037/bd4964ea-884e-11e5-928a-3e3c84f37662.png"></td>
<td><a link="https://github.com/pyblish/pyblish-linux">pyblish-linux</a></td>
<td></td>
<td>Officially supported modules and binaries for Linux (package)</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087037/bd4964ea-884e-11e5-928a-3e3c84f37662.png"></td>
<td><a link="https://github.com/pyblish/pyblish-osx">pyblish-osx</a></td>
<td></td>
<td>Officially supported modules and binaries for OSX (package)</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087037/bd4964ea-884e-11e5-928a-3e3c84f37662.png"></td>
<td><a link="https://github.com/pyblish/pyblish-x">pyblish-x</a></td>
<td>pyblish-win</td>
<td>Officially supported modules (package)</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish">pyblish</a></td>
<td>pyblish-x</td>
<td>Core module, the heart of Pyblish.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-qml">pyblish-qml</a></td>
<td>pyblish-x</td>
<td>User interface module, the face of Pyblish.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-tray">pyblish-tray</a></td>
<td>pyblish-x</td>
<td>The Pyblish control panel</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-rpc">pyblish-rpc</a></td>
<td>pyblish-x</td>
<td>Communication bridge between the core and user interface of Pyblish.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-integration">pyblish-integration</a></td>
<td>pyblish-x</td>
<td>Supporting module for [pyblish-rpc][]</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-maya">pyblish-maya</a></td>
<td>pyblish-x</td>
<td>Integration module for Autodesk Maya</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-houdini">pyblish-houdini</a></td>
<td>pyblish-x</td>
<td>Integration module for SideFx Houdini</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-ci">pyblish-ci</a></td>
<td></td>
<td>Continuous integration server for Pyblish.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087051/d2fb2620-884e-11e5-940a-f57c3265f8fc.png"></td>
<td><a link="https://github.com/pyblish/pyblish-event">pyblish-event</a></td>
<td></td>
<td>Reference implementation of a cloud-based event monitor.</td>
</tr>

</tbody>
</table>

[module]: https://cloud.githubusercontent.com/assets/2152766/11087037/bd4964ea-884e-11e5-928a-3e3c84f37662.png

### 3.3. Source structure

The following is the file structure of the core Pyblish module.

<table>
<thead>
<th></th>    <th>File</th>    <th>Description</th>
</thead>
<tbody>
<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087071/f1c6172c-884e-11e5-87b2-d2f502a01961.png"></td>
<td><a link="https://github.com/pyblish/pyblish/tree/master/pyblish/plugins">plugins</a></td>
<td>Default plug-ins.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087071/f1c6172c-884e-11e5-87b2-d2f502a01961.png"></td>
<td><a link="https://github.com/pyblish/pyblish/tree/master/pyblish/vendor">vendor</a></td>
<td>Third party dependencies (no external dependencies).</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/__init__.py">__init__.py</a></td>
<td>Global, private variables.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/__main__.py">__main__.py</a></td>
<td>Making this package executable.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/api.py">api.py</a></td>
<td>The developer facing interface to Pyblish.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/cli.py">cli.py</a></td>
<td>Command-line interface, written with the Click support library.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/compat.py">compat.py</a></td>
<td>Backwards and forward-compatibility features.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/error.py">error.py</a></td>
<td><strike>Deprecated</strike></td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/lib.py">lib.py</a></td>
<td>Helper functions used across surrounding Python modules.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/main.py">main.py</a></td>
<td><strike>Deprecated</strike></td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/util.py">util.py</a></td>
<td>Convenience module for publishing via scripting.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/version.py">version.py</a></td>
<td>The semantic version of this Pyblish module.</td>
</tr>
</tbody>
</table>

And finally, the most relevant files in terms of developing for Pyblish:

<table>
<thead>
<th></th>    <th>File</th>    <th>Description</th>
</thead>
<tbody>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/plugin.py">plugin.py</a></td>
<td>Plug-in definition, registration and discovery on disk.</td>
</tr>

<tr>
<td><img src="https://cloud.githubusercontent.com/assets/2152766/11087076/fb636500-884e-11e5-836c-a78d116dd9d5.png"></td>
<td><a link="https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py">logic.py</a></td>
<td>The brains of processing.</td>
</tr>

</tbody>
</table>

### 3.4 Processing Pipeline

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