# Implementing a feature

The feature we will implement will be adding the ability for Pyblish to support an [Instance][] with multiple families.

### Defining your goals

Any change to Pyblish must..

- Have a corresponding issue where the problem or feature is clearly defined
- Pass the tests of its corresponding module

For **a new feature**, a change must also..

- Have a corresponding topic on the forums for it to be discussed
- Be well defined in its corresponding issue.
 
Well defined means it has..

- Clearly stated goals
- A motivation
- An example implementation (when relevant)
- Notes on related research
- Example use-cases

And here is an example of a **bugfix** - [Plug-in filepath can't be retrieved][2]<br>
Here is an example of a new **feature** - [Instance multi-family support][1]<br>

[Instance]: https://github.com/pyblish/pyblish.api/wiki/Instance
[1]: https://github.com/pyblish/pyblish/issues/231
[2]: https://github.com/pyblish/pyblish/issues/229

### Zoning in

The next step is part of what is commonly referred to as test-driven development and is the point at which we write a test for a feature that doesn't yet exist or bugfix that hasn't yet been fixed.

This test will and should fail, up until the point at which we are finished. And that's how we know we're done. Simple.

### Writing tests

We'll be implementing the feature referenced above, to add support for multiple families within a single instance. A major overhaul to how plug-ins are created and how TDs are able to think about their publishing pipeline.

Tests are naked functions that either pass silently or throw an exception.

```python
def test_something():
  """Something works"""
  assert 1 < 2
  assert len("moon") == 4
```

By convention, tests start with `test_` followed by a short summary of what it tests for. The docstring appears in the terminal when run and should provide one line description of what it tests for, along with an optional long description should you need it.

Tests are generally many and simple, as opposed to few and complex. When writing a test, you will want to touch as few parts of a system as possible, but no fewer, such that the test will last once surrounding features are altered.

Let's write our test now.

```python
# pyblish/tests/test_plugins.py

@with_setup(lib.setup_empty, lib.teardown)
def test_multi_families():
    """Instances with multiple families works well"""

    count = {"#": 0}

    class CollectInstance(pyblish.api.Collector):
        def process(self, context):
            instance = context.create_instance("MyInstance")
            instance.data["families"] = ["geometry", "human"]

    class ValidateHumans(pyblish.api.Validator):
        families = ["human"]

        def process(self, instance):
            assert "human" in instance.data["families"]
            count["#"] += 10

    class ValidateGeometry(pyblish.api.Validator):
        families = ["geometry"]

        def process(self, instance):
            assert "geometry" in instance.data["families"]
            count["#"] += 100

    for plugin in (CollectInstance, ValidateHumans, ValidateGeometry):
        pyblish.api.register_plugin(plugin)

    pyblish.util.publish()

    assert count["#"] == 110, count["#"]
    
```

The test creates three plug-ins that assume that instances support multiple families. The decorator appends some helper functionality to the test, in this case allowing us to alter the currently registered plug-ins without affecting global state.

Running this test right away will fail. Let's try it out.

Open up a terminal, and type this in.

```bash
cd pyblish
python run_testsuite.py
...
Simple plug-ins process instances as usual ... ok
Simple plug-ins defaults to running *before* SVEC ... ok

======================================================================
FAIL: Instances with multiple families works well
----------------------------------------------------------------------
Traceback (most recent call last):
  File "C:\Users\marcus\Dropbox\AF\development\marcus\pyblish\pyblish\pyblish\vendor\nose\
case.py", line 197, in runTest
    self.test(*self.arg)
  File "C:\Users\marcus\Dropbox\AF\development\marcus\pyblish\pyblish\tests\test_plugin.py
", line 500, in test_multi_families
    assert count["#"] == 110, count["#"]
AssertionError: 0

----------------------------------------------------------------------
Ran 121 tests in 0.588s

FAILED (failures=1)
```

### Fixing the test

Now that we have a problem, let's find a solution.

Here is the function responsible for determining whether a set of instances are compatible with a given plug-in.

```python
# pyblish/logic.py
def instances_by_plugin(instances, plugin):
    """Return compatible instances `instances` to plugin `plugin`

    Arguments:
        instances (list): List of instances
        plugin (Plugin): Plugin with which to compare against
    Returns:
        List of compatible instances
    Invariant:
        Order of remaining plug-ins must remain the same

    """

    compatible = list()

    for instance in instances:
        family = instance.data["family"]
        if any(x in plugin.families for x in (family, "*")):
            compatible.append(instance)

    return compatible
```

We must augment this function. So let's upgrade this.

```python
    compatible = list()

    for instance in instances:
        family = instance.data["family"]
        if any(x in plugin.families for x in (family, "*")):
            compatible.append(instance)

    return compatible
```

To this.

```python
    compatible = list()

    for instance in instances:
        family = instance.data["family"]
        if any(x in plugin.families for x in (family, "*")):
            compatible.append(instance)
        elif set(plugin.families) & set(instance.data.get("families", [])):
            compatible.append(instance)

    return compatible
```

This comparison will pass only if there are identical families present in both the plug-in and instance. Which is exactly what we want.

Now we can run the tests to see if it does what we expect it to.

```bash
cd pyblish
python run_testsuite.py
...
Instances with multiple families works well ... ok
...

----------------------------------------------------------------------
Ran 121 tests in 0.540s

OK
```

Hurrah!

### Busywork

The final step is to increment the version of your respective repository and make a note of what has changed in it's changelog.

The version is located within a `version.py` file at the Python package-level of the repository.

![image](https://cloud.githubusercontent.com/assets/2152766/11124670/14024562-895e-11e5-8bb8-f1fceed34baa.png)

But before incrementing, make sure there isn't already a new version "in the oven". This is best found out on the forums or on chat rooms. For changes made to a version not yet released (i.e. it doesn't yet have a GitHub Release) you *do not* need to increment it.

In other cases, you do and here's how.

- http://semver.org

Pyblish loosely follows the Semantic Versioning system, more information about it can be found on it's deducated website.

- PATCH increments are generally used for most changes that doesn't involve an announcement and tutorials on the new change.
- MINOR changes are used for large but backwards compatible changes and new features.
- MAJOR is reserved for large, potentially backwards incompatible changes.

You should in general aim for backwards compatibility, but sometimes a clean slate safe-guards future growth.

Finally, it's time to make a note in the `CHANGES` under the corresponding version. If you incremented, you will need to make a new entry.

![image](https://cloud.githubusercontent.com/assets/2152766/11125256/d89cd03e-8960-11e5-8e8b-4dfaf59e665e.png)

The CHANGELOG looks something like this.

```bash
pyblish Changelog
=================

This contains all major version changes between pyblish releases.

Version 1.2.2
-------------

- Added support for instances with multiple families (#231)
```

It's customary to include a reference to the corresponding issue, here that is [#231][].

[#231]: https://github.com/pyblish/pyblish/issues/231

#### Committing

When making commits, here are a few guidelines for you to think about.

- Keep commits focused. Don't touch code not relevant to 1 specific problem at a time.
- Keep commits few. One meaningful commit is better than three haphazard ones.
- Describe your commits. You should be able to browse a listing of commits and understand what is changing without diving into the code.
- 
<div class="modified-date">{{ file.mtime }}</div>