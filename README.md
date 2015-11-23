![image](https://cloud.githubusercontent.com/assets/2152766/11151115/25ac4ee6-8a23-11e5-9b48-01c3e8778d27.png)

A maintainers and feature developers guide to Pyblish.

**See also**

- [Tutorial](http://forums.pyblish.com/t/learning-pyblish-by-example/108?u=marcus)
- [Discussion forums](http://forums.pyblish.com/t/developer-guide)

<br>
<br>
<br>

### Goal

Once finished reading this article, you should feel confident in maintaining and/or extending Pyblish at any of the following levels of granularity.

- A) *Source code optimisations* - Such as spelling errors, removing unused code etc.
- B) *Bug fixes* - Including familiarity with Git and continuous integration for stable code.
- C) *Minor and major features* - To increment the (semantic) version.

<br>
<br>
<br>

### Overview

Pyblish is an analytics and automation framework developed in Python for the visual effects and games industries in order to detect and resolve problems as early in the pipeline as possible.

It was designed to combat issues encountered in the day-to-day development of content involving chains of artists working on disparate parts of the same end result. Where a minor fault early on can have a significant impact later and be that much more expensive to fix.

##### Modularity

The framework is built as a series of "modules".

![image](https://cloud.githubusercontent.com/assets/2152766/11087003/82a4d57c-884e-11e5-8b3c-7f89e6cc1f5e.png)

Each module independently developed and represented by a repository hosted on the online collaboration platform GitHub. There, each module maintains their own documentation, version, issues tracking, stars and wiki.

Modules either define, extend or integrate Pyblish at some level.

At the time of this writing, there are 40 actively developed repositories in the [Pyblish Organisation on GitHub][2].

##### Packages

The inherent flexibility of modules maintaining their own version comes at a cost; maintaining a working state between all possible combinations of versions.

```yaml
40 modules ^ 50 versions each = 1.26e80
```
That's 1.26 with 80 zeroes behind it. A.k.a. "Not Easy To Maintain™".

To combat this, some modules form so-called "packages".

![image](https://cloud.githubusercontent.com/assets/2152766/11087014/920cb84a-884e-11e5-9c96-16e5a63a2160.png)

Packages *group* a number of other modules into a combination of versions known to work together. For example, version 0.2.0 of [pyblish-x][] consists of the following modules and versions.

- [pyblish][] v1.2.1
- [pyblish-qml][] v0.3.2
- [pyblish-rpc][] v0.1.4
- [pyblish-integration][] v0.1.1
- ...

The packages then provide both the user with a foundation for communicating which combination a problem of feature request is with and the developer with an ability able to surgically dedicate more resources into supporting only a few select versions of a single module, as opposed to the otherwise exponential amount of combinations.

> "I'm using Pyblish X version 0.2 and I'm having this problem."

##### Integrations

Pyblish tackles the human "problem" of not being adequately precise or consistent with respect to what is required during the production of computer graphics. As such it can't be limited by the specific software you run.

To facilitate this, the Pyblish core is a software-agnostic Python library capable of running in any software with support for Python 2.6 or 2.7 (soon also Python 3), such as Autodesk Maya 2008 and upwards, Nuke, Modo, Fusion and Photoshop to name a few. Including those with *no* support for Python, via standalone publishing.

Some examples include..

- [Pyblish for Maya][maya]
- [Pyblish for Houdini][houdini]
- [Pyblish for Nuke][nuke]
- [Pyblish for Hiero][hiero]

Integrations are simple and typically boils down to a few lines of code with optional extras. Developers are encouraged to integrate hosts on their own via the dedicated [Pyblish Integration][3] module.

**See also**

- Integration Guide (coming soon)

##### Extensions

Because the problems you face during production are likely unique, Pyblish cannot make too many assumptions about how you will attempt to solve them.

Instead, Pyblish delivers the "shell" within which you define your problem domain via the power of Python and the supplied functionality of the Pyblish framework. To jumpstart your journey into breaking down and defining your pipeline, a number of modules are provided that gather common problems found in a particular domain; such as full computer graphics advertisements developed at a mid-sized company or a blend between live-action and visual effects produced at a larger facility.

These modules make assumptions about the way you work that you can either employ or be inspired by.

Some examples include..

- [Pyblish Magenta][magenta]
- [Pyblish Napoleon][napoleon]

Developers are encouraged to define, document and deploy their own unique workflows in the form of extensions, and share them with the world. This way, the efforts made by one can benefit all which ultimately makes our industry a better place to be.

<br>

**Reading on GitHub?**

- [Continue Reading](https://pyblish.gitbooks.io/developer-guide/content/workflow.html)

<div class="modified-date">{{ file.mtime }}</div>

[maya]: https://github.com/pyblish/pyblish-maya
[houdini]: https://github.com/pyblish/pyblish-houdini
[nuke]: https://github.com/pyblish/pyblish-nuke
[hiero]: https://github.com/pyblish/pyblish-hiero
[magenta]: https://github.com/pyblish/pyblish-magenta
[napoleon]: https://github.com/pyblish/pyblish-napoleon
[pyblish-qml]: https://github.com/pyblish/pyblish-qml
[pyblish-rpc]: https://github.com/pyblish/pyblish-rpc
[pyblish]: https://github.com/pyblish/pyblish
[pyblish-integration]: https://github.com/pyblish/pyblish-integration
[pyblish-x]: https://github.com/pyblish/pyblish-x

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