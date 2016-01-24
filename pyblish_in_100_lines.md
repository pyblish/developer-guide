# Pyblish in 100 lines

Pyblish is a tiny framework. Even though it consists of over 40 individual Git repositories, only one of them represents the actual mechanism and is tiny enough to fully understand.

What better way for me to reflect, and for you to understand this, than to demonstrate how it all fits together in a single block of code?

| **Part**                  | **Description**
|:--------------------------|:-------------
| [Code](#code)             | 100 lines of standalone code representing the Pyblish core.
| [Breakdown](#breakdown)   | Highlights and describes some of the higher level lessons to take away from this article.

<br>
<br>
<br>

### Code

The following represents Pyblish at it's core, no fuzz. It runs directly in Python with no dependencies, in ~100 lines of code, including comments.

- [Code as GitHub Gist](https://gist.github.com/mottosso/b923f5b29dfceaa6d86c)

**Including**

- An accurate representation of module and class hierarchy
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
import types

# Mocked Pyblish module
pyblish = types.ModuleType('pyblish')
pyblish.api = types.ModuleType('api')
pyblish.api.CollectorOrder = 0
pyblish.api.ValidatorOrder = 1
pyblish.plugin = types.ModuleType('plugin')
pyblish.logic = types.ModuleType('logic')


# Ignore this class
class _Plugin(object):
    def __init__(self):
        self._records = list()
        self.log = type('Logger', (object,), {})()
        self.log.info = lambda text: self._records.append(text + '\n')


# In place of Plugin and CVEI superclasses, two distinct types are defined.
# Each has a unique, explicit behaviour, as opposed to the implicit behavior
# present in the current CVEI types.
class ContextPlugin(_Plugin):
    pass


class InstancePlugin(_Plugin):
    pass


# Example plugins
class CollectInstances(ContextPlugin):
    # The order is equally explicit and assigned
    # either via static, named numbers, or as usual
    # via arbitrary numbers. Sorting remains unchanged
    order = pyblish.api.CollectorOrder

    def process(self, context):
        context.append('instance1')
        context.append('instance2')
        self.log.info('Processing context..')


class ValidateInstances(InstancePlugin):
    order = pyblish.api.ValidatorOrder

    def process(self, instance):
        self.log.info('Processing %s' % instance)

plugins = [ValidateInstances, CollectInstances]
plugins = sorted(plugins, key=lambda item: item.order)

# plugin.process is greatly simplified.
# Note the disappearance of the provider.
def process(plugin, **kwargs):
    print('individually processing %s' % kwargs.values()[0])

    result = {
        'plugin': plugin,
        'item': None,
        'error': None,
        'records': list(),
        'success': False
    }

    try:
        plugin = plugin()
        plugin.process(**kwargs)
    except Exception as e:
        result['success'] = False
        result['error'] = e

    result['records'] = plugin._records

    return result

pyblish.plugin.process = process


# logic.process is greatly simplified
def process(plugins, context):
    print('logic.process running..')

    for plugin in plugins:
        print('Processing %s' % plugin)

        # Run once
        if issubclass(plugin, ContextPlugin):
            yield pyblish.plugin.process(plugin, context=context)

        # Run once per instance
        if issubclass(plugin, InstancePlugin):
            for instance in context:
                yield pyblish.plugin.process(plugin, instance=instance)

pyblish.logic.process = process


# Example usage
context = list()
processor = pyblish.logic.process(plugins, context)
results = list(processor)
print(results)
```

<br>
<br>
<br>

### Breakdown

Once you've run the above code, absorbed it's output, let's turn our attention to some of the novelties within.

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