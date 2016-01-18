# Suggestions for Improvements

Here are a few things that I personally think could be improved.

- [logic.process][] is too complex.
- [logic.process][] currently does not take into account the *next* plug-in to be processed, instead it inaccurately considers the last plug-in to be the next plug-in, causing the GUI to visualise the wrong plug-in.
- [logic.test][] is too complex. It's questionable whether it ever needs to be extended or whether it has been overengineered.
- Dependency Injection is too complex with very little gain.
- Dependency Injection changes the behaviour of plugin.process, such that passing `instance` causes it to behave differently than when *not* passing it.
- Python 3 support

[logic.process]: https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py#L51
[logic.test]: https://github.com/pyblish/pyblish/blob/master/pyblish/logic.py#L17

Below are some further in-depth topics of suggested improvements.

- [Simplicity](#simplicity)
- [Brittleness](#brittleness)
- [Iterator and Responsibility](#iterator-and-responsibility)
- [`ContextPlugin` & `InstancePlugin`](#ContextPlugin--- InstancePlugin)

<br>
<br>
<br>

### Simplicity

The greatest design challenge currently, is the implicit behavioral difference in passing `context` and passing `instance` to the `process()` method of a plug-in.

Passing only `context` causes a plug-in to process only the context, but passing `instance` causes it to process both the context *and* each currently available instance.

Both the implementation and behavior is both surprising and un-Pythonic. The goal is to make this more Pythonic and less surprising.

<br>
<br>
<br>

### Brittleness

Pyblish, like any program, consists of a lot of files. However in the case of Pyblish, the installation is a match 1-1 with the development environment, and the development environment is designed for flexibility - not robustness. Because of this, the installed copy of Pyblish is succeptible to hard-to-debug problems.

For example, because each component of Pyblish is a Git repository, it is technically possible to run `git pull` on an individual repository. However, due to various reasons (permissions, disk space, time of day or color of the sky) some files may either not get updated or some files may get left behind that should really have gotten removed.

An improvement would be to increase robustness; either by reducing the amount of files, or somehow better safeguarding files; possible by removing the Git history. Another option might be to compile/pack as many files as possible into as few binaries as possible. (Using zip-archives as modules?)

<br>
<br>
<br>

### Iterator and Responsibility

Currently, the Pyblish QML GUI lacks the ability to tell you which plug-in is currently running. It simply can't know this, because the decision isn't made by the GUI, it's made by the core library to which the GUI is a slave. So, a request is sent "Hey, publish *these* plug-ins using *this* context" to which the Pyblish core responde "Ok, I'll let you know how it goes".

This is no good. The GUI is a controller, it should be able to make decisions about what to do. In that way, you are able to make those decisions. Therefore, it is imperative that the decision be put into the hands of the GUI such that the GUI can then offer the choice to you, and in effect gain the ability to accurately represent what is happening.

How can we achieve this?

An iterator. The iterator's responsibility will be to simply yield the next reasonable pair of plug-in/item, `item` depending on which Instance, or Context, is currently running and `plug-in` depending on whose turn it is to process.

That's it. That's where the core stops being helpful. The rest is up to the GUI. With a few exceptions.

The reason all of this isn't implemented in the GUI right away, is because the GUI isn't the only interface to publishing. You also have `pyblish.util.publish()` and the CLI. Anything we implement in the GUI, will have to be duplicated in each of those, and in each of the future interfaces that may be developed.

So a balance must be struck. Currently, that balance is quite accurate. Like when you get up on one wheel with your bicycle and monitor whether you are pedaling slightly too much, or slightly too little and you adjust. But the bike is leaning too much forwards and you compensate by pedaling faster. It won't last.

More will have to be implemented (i.e. duplicated) within each interface in order for each interface to regain usability, stability, maintainability and for the core to regain extendibility.

<br>
<br>
<br>

### `ContextPlugin` & `InstancePlugin`

There are two distinct modes of operation in Pyblish that hasn't gotten the individal spotlight they each deserve. Instead they have been crammed into a complex cocktail of engineering; dependency injection.

The end result is less typing (and a slick interface, if I may say so myself) at the expense of higher cognitive load and difficult debugging.

Split it up.

Make a point that each are separate and must be treated as such.

```python
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
```

<div class="modified-date">{{ file.mtime }}</div>