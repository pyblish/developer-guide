# Workflow

Pyblish is designed to be flexible whilst providing as much aid as possible.

Aid is delivered in the form of API and user interface. The API provides the developer with a level of abstraction within which pipeline requirements are to be described. Once described, Pyblish is able to automate and visualise a lot of things for you and your artists.

**Note** It is assumed you have already completed [Learn Pyblish by Example][].

### Development strategy

"CVEI" is an architectural pattern for describing a pipeline in terms of publishing and is an acronym for Collection, Validation, Extraction and Integration.

It is described and executed in the following order.

1. **Collection** is the point at which data from a given environment is parsed and stored for processing by the three subsequent steps. It is here where you, the developer, identify the logical sets of data within an artists working scene that is to become physical files on disk.
2. **Validation** is where the collected data is analysed for correctness. Failing validation means one or more problems have been found that require an artist's attention and not until the problems have been resolved can processing continue.
3. **Extraction** is synonymous with export; it means to physically extract the collected data out in an application-dependent manner such that it may be persistently stored on disk, database or other form of store.
4. **Integration** is where the extracted files are finally organised and named according to convention, and where additional metadata is created, emitted and/or submitted, such as registering the event in a production tracking system.

It's important to point out that these steps are mere guidelines and that Pyblish triggers whatever you throw at it in any order. But should you choose to mold your definition and understanding of your publishing pipeline in this way you enable yourself and those around you to take advantage of and to share the knowledge and vocabulary of other Pyblishers.

### Plug-ins

The way you express yourself through CVEI is through the use of "plug-ins".

The Pyblish API is based around the notion that you write functionality that Pyblish then runs. The order in which plug-ins run is then something you control. Plug-ins are generally written by the technical director, but are also flexible enough to enable the artist to add functionality on-the-fly by simply registering it with Pyblish at run-time.

**Example**

```python
import pyblish.api

class MyPlugin(pyblish.api.Plugin):
    def process(self):
        print("Every plug-in subclasses either Plugin "
              "or subclass of Plugin")
```

### Communication

It is the technical directors responsibility to identify and program the various conventions found in a pipeline so as to safeguard an artist from producing content that might eventually lead to problems further down the pipeline. The technical director then communicates his definitions with the artist via 3 primary mechanisms.

1. Plug-in Documentation
1. Log messages
1. Exception messages

The documentation generally describes why a particular plug-in exists and what it is meant to protect. It is generally as thorough as possible, acting as a first line of defence between an artist encountering a problem and the artist having to go look for help about how to fix it.

Log messages provide the artist with hints as to what is currently happening and what has happened during the processing of the processed plug-ins. They are meant to leave a "trail of breadcrums" with which an artist might narrow down any particular problem. A log message can get as specific as needs be, and is relative to an artists current working scene. Including things such as the actual names of nodes or attributes were analysed.

Finally, the exception messages signal to the artist that a plug-in has failed. This can either be during validation, in which case processing does not proceed to extraction.

**Example**

The following is an example of each of the three mechanisms.

```python
import pyblish.api

class MyPlugin(pyblish.api.Plugin):
    """This docstring will be available as via the user interface"""
    def process(self):
        # Standard logging levels are available as you would expect.
        self.log.info("This is a general progress message for artists.")
        self.log.debug("This is a progress message mainly intended for the developer.")

        raise Exception("Exceptions signal the failure of a plug-in.")
        assert False is True, "Any kind of Python exception is ok."
```

### Context

Pyblish provides a level of orchestration amongst your plug-ins via the use of a shared "context".

This object is created at the start of processing of the first plug-in and is passed along from one plug-in to the next. It is via the context that plug-ins may share and exchange information with one another. You can think of the context as a global variable with a lifetime spanning between the start and end of a single publish.

**Example**

The following is an example of how to access the context.

```python
import pyblish.api
context = pyblish.api.Context()
```

The context is subclassed from a standard Python list and can be thought of as such, with the addition of a `.data` dictionary in which arbitrary data may be stored and shared amongst plug-ins.

```python
assert isinstance(context, list)
context.data["globalData"] = 10
```

### Instance

The final and most important mechanism for orchestrating plug-ins is the [Instance][].

Instances are generally created during Collection and represents the inverse of a file on disk. It is the encapsulation of the subset of data within a work area that can potentially become a file. That can be a mouthful, so here's another way to think of it; when you import a file into your scene, the resulting data is an instance.

The general idea is for you to identify and articulate the parts of your work area that is to become a file, such as a character rig or animation, collect relevant information about this Instance and finally extract and integrate it into your pipeline as one physical file.

Technically, Instances are also subclassed from the standard Python list in order to facilitate either a hierarchy of Instances or arbitrary data that may be better represented as individual items - such as Maya nodes - and also provide access to it's own `.data` dictionary.

**Example**

[Instance][]s are created the same way as the [Context][], except it must be given a name.

```python
import pyblish.api
instance = pyblish.api.Instance(name="MyInstance")
```

It is generally always a child of the [Context][], like this.

```python
import pyblish.api
context = pyblish.api.Context()
instance = pyblish.api.Instance(name="MyInstance", parent=context)

# This is a common-enough pattern, that there's a convenience
# function for it which does exactly what we did above.
instance = context.create_instance("MyInstance")
```

### Dependency injection

You generally never have to worry about instantiating the [Context][]. Instead, the [Context][] is instantiated for you at the start of processing and is delivered to plug-ins via something called "dependency injection".

Dependency injection basically means that whatever is required is provided. In this case, when the [Context][] is required in a plug-in, it is provided as a variable. 

The end result is a single, coherent method for you to override in any and all of your plug-ins.

**Example**

```python
import pyblish.api

class MyContextPlugin(pyblish.api.Plugin):
    def process(self, context):
        print("I have access to the Context..")


class MyInstancePlugin(pyblish.api.Plugin):
    def process(self, instance):
        print("..I have access to the *current* Instance..")
        
        
class MyMultiPlugin(pyblish.api.Plugin):
    def process(self, context, instance):
        print("..and I have access to both the Instance and Context.")
```

*Dependency injection in Pyblish was inspired by how it is used and implemented in AngularJS. - [Reference](https://github.com/angular/angular.js/wiki/Understanding-Dependency-Injection)*

### Conditions

A final thing to mention about instances is conditions.

![image](https://cloud.githubusercontent.com/assets/2152766/11087503/bfc6b5fc-8852-11e5-96ba-1d0a4dd92941.png)

In Pyblish, Instances are "paired" or "associated" with one or more plug-in via what is referred to as "families". An Instance with a family matching one of the supported families of a plug-in is considered related and will be sent for processing.

This is an incredibly powerful and unique aspect of Pyblish and is the one with which you are able to design vast and complex hierarchies and dependencies amongst tens, hundreds even thousands of plug-ins whilst still retaining that surgical precision needed to pinpoint and massage the tiniest problem domains.

**Example**

The following is an example of two Instances being identified and "collected". The instances are then passed onto subsequent plug-ins, but only one of the instances is a match with one of the plug-ins, as defined by it's "family" and the "families" each plug-in supports.

```python
import pyblish.api

items = ["john.person", "door.prop"]

class CollectInstances(pyblish.api.Plugin):
  order = 10

  def process(self, context):
    for item in items:
      name, suffix = item.split(".")
      context.create_instance(name, family=suffix)

class PrintPersons(pyblish.api.Plugin):
  order = 20
  families = ["person"]

  def process(self, instance):
    print("Person is: %s" % instance)

class PrintProps(pyblish.api.Plugin):
  order = 20
  families = ["prop"]

  def process(self, instance):
    print("The prop is: %s" % instance)

pyblish.api.register_plugin(CollectInstances)
pyblish.api.register_plugin(PrintPersons)
pyblish.api.register_plugin(PrintProps)

import pyblish.util
pyblish.util.publish()
# The person is "john"
# The prop is "door"
```

[Instance]: https://github.com/pyblish/pyblish.api/wiki/Instance
[Context]: https://github.com/pyblish/pyblish.api/wiki/Context
[Learn Pyblish by Example]: http://forums.pyblish.com/t/learning-pyblish-by-example