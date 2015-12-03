# Suggestions for Improvements

Here are a few things that I personally think could be improved.

- logic.process is too complex.
- logic.process currently does not take into account the *next* plug-in to be processed, instead it inaccurately considers the last plug-in to be the next plug-in, causing the GUI to visualise the wrong plug-in.
- logic.test is too complex. It's questionable whether it ever needs to be extended or whether it has been overengineered.
- Dependency Injection is too complex with very little gain.
- Dependency Injection changes the behaviour of plugin.process, such that passing `instance` causes it to behave differently than when *not* passing it.
- Python 3 support

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

An improvement would be to increase robustness; either by reducing the amount of files, or somehow better safeguarding files; possible by removing the Git history. Another option might be to compile/pack as many files as possible into as few binaries as possible.

<div class="modified-date">{{ file.mtime }}</div>