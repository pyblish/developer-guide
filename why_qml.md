
# Why QML?

TLDR; In order to remain competitive.

[pyblish-qml][] is the officially supported user interface for Pyblish and is written in Python 2.7 and QML 2 with PyQt5.

QML is a declarative language developed by The Qt Company to develop OpenGL accelerated graphical user interfaces. QML was originally inspired by CSS and aims to simplify the development of graphical user interfaces by decoupling code written for visuals from code written for business logic, enabling artists to step in with less of a programming background, and programmers to worry less about cosmetics.

QML was chosen as graphical framework for Pyblish due to (1) being the successor to the traditional Widgets, (2) for its novelty and speed in terms of how software is developed but most importantly in order to (3) as an individual/small company be able to compete with big organisations who are in most likelyhood stuck writing widgets for the foreseeable future.

### Current state of the art

Many integration endpoints, such as Autodesk Maya, facilitate integration by providing the developer with an ability to *mix* custom software with the host. For example, one can both augment and append to existing widgets within the host itself.

This has a few pros.

- Effortless deployment.
- Effortless installation.
- Shared memory

It also comes with a few cons.

- Outdated software
- Version discrepancies
- Custom implementations
- Sharing of weaknesses
  - Such as having threads locked at the whim of the host
  - Not facilitating asynchronism, leading to a jittery end-user experience.
  - Not being able to introspect the custom software environment; the provider is in charge and it is proprietary.

<br>

For hack-and-slash software, written in the heat of production, the provided integration is indispensable. Enabling the rapid development of tools otherwise not possible. For software meant to last, the future is now.

<div class="modified-date">{{ file.mtime }}</div>

[pyblish-qml]: https://github.com/pyblish/pyblish-qml
