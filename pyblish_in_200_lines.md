# Pyblish in 200 lines

> Unfinished

Pyblish is a tiny framework. Even though it consists of over 40 individual Git repositories, the actual mechanism is small and nimble.

What better way for me to reflect this and for you to understand this than to illustrate how it all fits together in a single block of code?

<br>
<br>
<br>

### 200 lines

The following represents Pyblish at it's core, without fuzz.

**Including**

- An accurate representation of actual module and class hierarchy
- The dependency injection mechanism
- Processing
- CVEI
- Logging
- The `results` dictionary

**Excluding**

- .data
- Stopping on failed validation
- Actions


```python
"""Core behavior of Pyblish

Included:
Missing:

"""

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


# The dependency-injection mechanism.
# Available arguments to a plug-ins `process()` method
# as injected here and later passed to the function via `invoke()`.
class Provider(object):
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


# Ordering remains unchanged
plugins = [ValidateInstances, CollectInstances]
plugins = sorted(plugins, key=lambda item: item.order)


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


def process(plugins, context):
    print("logic.process running..")

    for plugin in plugins:
        print("Processing plug-in: %s" % plugin)

        # Run once
        if not plugin.__instanceEnabled__:
            yield pyblish.plugin.process(
                plugin=plugin,
                context=context,
                instance=None)

        # Run per instance
        else:
            for instance in context:
                yield pyblish.plugin.process(
                    plugin=plugin,
                    context=context,
                    instance=instance)

pyblish.logic.process = process


# Example usage
context = list()
processor = pyblish.logic.process(
    plugins=plugins,
    context=context
)

print("Starting..")
results = list()
for result in processor:
    results.append(result)

print("\nFinished, printing results")
print(json.dumps(results, indent=4))

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

### Key points

1. There are 2 `process()`
2. Notice how `Provider` is instantiated, injected and then used.