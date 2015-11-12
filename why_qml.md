
# 4.1 Why QML?

TLDR; In order to remain competitive.

[pyblish-qml][] is the officially supported user interface for Pyblish and is written in Python 2.7 and QML 2 with PyQt5.

QML is a declarative language developed by The Qt Company to develop OpenGL accelerated graphical user interfaces. QML was originally inspired by CSS and aims to simplify the development of graphical user interfaces by decoupling code written for visuals from code written for business logic, enabling artists to step in with less of a programming background, and programmers to worry less about cosmetics.

QML was chosen for being the successor to the traditional Widgets, for its novelty and speed in terms of how software is developed but most importantly in order to, as an individual/small company, be able to compete with big organisations who may share my goals but who are stuck writing widgets for the foreseeable future.

All roads point to PySide and Qt 4 being the most lucrative development framework for film and games, but if one is to inspire change and take any kind of lead, one must "think different".

Many integration endpoints, such as Autodesk Maya, facilitate integration by providing the developer with an ability to *mix* custom software with the host. For example, one can both augment and append to existing widgets within the host itself.

This has many pros.

- Custom software can get by with zero dependencies, apart from the source code. This makes deployment effortless.
- Communication between host and custom software is equal compared with what already happening internally.
- Custom software will be more familiar to the end-user who may in turn wish to extend it further.

However it doesn't come without cons.

- Not all software provides this level of integration, splitting custom software into two parts.
- Each provider provides it differently, meaning bugs appearing in one host may not appear elsewhere, further dividing the codebase.
- Custom software shares the weaknesses of its provider.
  - This includes things such as having threads locked at the whim of the provider.
  - Not facilitating asynchronism, leading to a jittery end-user experience.
  - Not being able to introspect the custom software environment; the provider is in charge and it is proprietary.
- Finally, custom software being limited to past trends and unable to benefit from new developments in the field, impairing your ability to compete.

For hack-and-slash software, written in the heat of production, the provided integration is indispensable. Enabling the rapid development of tools otherwise not possible. For software meant to last, the future is now. Literally. In that one must consider how to ensure survival an ever-changing environment.
