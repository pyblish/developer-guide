# Pyblish in 200 lines

Pyblish is a tiny framework. Even though it consists of over 40 individual Git repositories, only one of them represents the actual mechanism and is tiny enough to fully understand.

What better way for me to reflect, and for you to understand this, than to demonstrate how it all fits together in a single block of code?

| **Part**                  | **Description**
|:--------------------------|:-------------
| [Code](#code)             | 200 lines of standalone code representing the Pyblish core.
| [Breakdown](#breakdown)   | Highlights and describes some of the higher level lessons to take away from this article.

<br>
<br>
<br>

### Code

The following represents Pyblish at it's core, no fuzz. It runs directly in Python with no dependencies, in 200 lines of code or less, including comments.

- [Code as GitHub Gist](https://gist.github.com/mottosso/9b28156f3b077e7048c8)

**Including**

- An accurate representation of module and class hierarchy
- The dependency injection mechanism
- Processing
- CVEI
- Logging
- The `results` dictionary

**Excluding**

- Stopping on failed validation
- .data[]
- Actions
- Plug-in discovery

> The """docstrings""" and `#` comments are for you.

```python
import json
import types
import inspect

# The core Pyblish package is mocked.
# These are all relevant members to processing.
pyblish = types.ModuleType("pyblish")
pyblish.api = types.ModuleType("api")
pyblish.plugin = types.ModuleType("plugin")
pyblish.plugin.process = None   # Implemented below
pyblish.plugin.Provider = None  # Implemented below
pyblish.logic = types.ModuleType("logic")
pyblish.logic.process = None    # Implemented below


# This metaclass hides certain (internal) aspects of plug-ins that
# the end-user (e.g. you) aren't intended to ever touch.
class MetaPlugin(type):
    def __init__(cls, *args, **kwargs):
        args_ = inspect.getargspec(cls.process).args
        cls.__instanceEnabled__ = "instance" in args_


class Plugin(object):
    __metaclass__ = MetaPlugin

    order = 0

    def __init__(self):
        # This illustrates how logging works and how logs are
        # stored and retrieved during processing.
        self._records = list()
        self.log = type("Logger", (object,), {})()
        self.log.info = self._records.append

    def process(self):
        # The default implementation does nothing
        # hence no need to call upon super().
        pass


# The CVEI superclasses, also does nothing but define a default order
Collector = type("Collector", (Plugin,), {"order": 0})
Validator = type("Validator", (Plugin,), {"order": 1})
Extractor = type("Extractor", (Plugin,), {"order": 2})
Integrator = type("Integrator", (Plugin,), {"order": 3})


class Provider(object):
    """The dependency-injection mechanism

    Available arguments to a plug-ins `process()` method
    are injected here and later passed to the function via `invoke()`.

    Example
        >>> def function(name):
        ...     print(name)
        ...
        >>> provider = Provider()
        >>> provider.inject("name", "Marcus")
        >>> provider.invoke(function)
        "Marcus"

    """

    def __init__(self):
        self._services = {}

    def inject(self, name, obj):
        self._services[name] = obj

    def invoke(self, func):
        args = self.args(func)
        return func(**{k: v for k, v in self._services.items()
                       if k in args})

    @classmethod
    def args(cls, func):
        return [a for a in inspect.getargspec(func)[0]
                if a not in ("self", "cls")]

pyblish.plugin.Provider = Provider


# Example plugins
class CollectInstances(Collector):
    def process(self, context):
        context.append("instance1")
        context.append("instance2")
        self.log.info("Processing context..")


class ValidateInstances(Validator):
    def process(self, instance):
        self.log.info("Processing %s" % instance)


# Ordering
plugins = [ValidateInstances, CollectInstances]
plugins = sorted(plugins, key=lambda item: item.order)


# Primary processing function.
# This function runs your plug-in and produces a `result` dictionary.
def process(plugin, context, instance):
    print("plugin.process running..")
    print("Individually processing \"%s\"" % (instance or "Context"))

    result = {
        "plugin": plugin.__name__,
        "instance": None,
        "error": None,
        "records": list(),
        "success": False
    }

    provider = pyblish.plugin.Provider()
    provider.inject("plugin", plugin)
    provider.inject("context", context)
    provider.inject("instance", instance)

    plugin = plugin()

    try:
        provider.invoke(plugin.process)
    except Exception as e:
        result["success"] = False
        result["error"] = e

    result["records"] = plugin._records

    return result

pyblish.plugin.process = process


# Logical processing function.
# This function delegates processing the appropriate plug-in/instance pair.
def process(func, plugins, context):
    print("logic.process running..")

    for plugin in plugins:
        print("Processing plug-in: %s" % plugin)

        # Run once
        if not plugin.__instanceEnabled__:
            yield func(
                plugin=plugin,
                context=context,
                instance=None)

        # Run per instance
        else:
            for instance in context:
                yield func(
                    plugin=plugin,
                    context=context,
                    instance=instance)

pyblish.logic.process = process


# Example usage
def publish():
    context = list()
    processor = pyblish.logic.process(
        func=pyblish.plugin.process,
        plugins=plugins,
        context=context
    )
    
    print("Starting..")
    results = list()
    for result in processor:
        results.append(result)
    
    print("\nFinished, printing results")
    print(json.dumps(results, indent=4))


if __name__ == "__main__":
    publish()
    # Starting..
    # logic.process running..
    # Processing plug-in: <class '__main__.CollectInstances'>
    # plugin.process running..
    # Individually processing "Context"
    # Processing plug-in: <class '__main__.ValidateInstances'>
    # plugin.process running..
    # Individually processing "instance1"
    # plugin.process running..
    # Individually processing "instance2"
    
    # Finished, printing results
    # [
    #     {
    #         "instance": null,
    #         "error": null,
    #         ...
```

<br>
<br>
<br>

### Breakdown

Once you've run the above code, absorbed it's output, let's turn our attention to some of the novelties in it.

1. There are 2 `process()` functions.
2. Notice how `Provider` is instantiated, injected and then used.

`plugin.process()` is the first runner up. It handles actually running your plug-in with either an [Instance][] or the full [Context][]. It isn't particularly interesting (except for maybe how it generates the [result][] dictionary, which is later [validated][1] by [json-schema][2] during interaction with [pyblish-rpc][]).

What is interesting however is the other `process()` function.

`logic.process()` is the first responder to publishing. It takes the available plug-ins and instances and delegates them in pairs `plugin.process`.

![image](https://cloud.githubusercontent.com/assets/2152766/11323548/867f318a-910c-11e5-8315-02b88322e0b4.png)

One thing to note here is that `logic.process` takes as its first argument a function to use for processing. This function is `plugin.process`, but only under ideal circumstances. It can't do that, for example, when running remotely, such as with [pyblish-qml][].

In the case of running remotely, `plugin.process` is replaced with an external function that is delegates the process to [pyblish-rpc][], which in turn runs `plugin.process` on the remote machine.

**See here for an example of how this works**

- [pyblish-qml/control.py](https://github.com/pyblish/pyblish-qml/blob/8d897c343fc7f31c868a6680a11f84dbe25a19f0/pyblish_qml/control.py#L566)

Dependency injection is the next item of interest. It isn't Pythonic, but it is very simple. Have a gander at its docstring and try it out for yourself to appreciate just how simple, yet flexible it is.

This mechanism was inspired by its use and implementation in the JavaScript framework AngularJS.

- [Reference](https://github.com/angular/angular.js/wiki/Understanding-Dependency-Injection)

[1]: https://github.com/pyblish/pyblish-rpc/tree/master/pyblish_rpc/schema
[2]: http://json-schema.org/
[result]: https://github.com/pyblish/pyblish.api/wiki/result
[Instance]: https://github.com/pyblish/pyblish.api/wiki/Instance
[Context]: https://github.com/pyblish/pyblish.api/wiki/Context
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
[pyblish-qml]: https://github.com/pyblish/pyblish-qml