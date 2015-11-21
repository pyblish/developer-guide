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
### Design Challenge

The greatest design challenge currently, is the implicit behavioral difference in passing `context` and passing `instance` to the `process()` method of a plug-in.

Passing only `context` causes a plug-in to process only the context, but passing `instance` causes it to process both the context *and* each currently available instance.

Both the implementation and behavior is both surprising and un-Pythonic. The goal is to make this more Pythonic and less surprising.

<div class="modified-date">{{ file.mtime }}</div>