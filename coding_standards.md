# Coding Standards

The Pyblish codebase has a very strict and consistent coding style. If you are to develop for Pyblish, it's a good idea to stick with it!

- A vast majority of these rules conform with [PEP8][1] and as such it is recommended that you familiarise yourself with it.
- For docstrings, you must follow the [Google Napoleon convention][2].
- For code complexity and health, you are advised to adhere to [pylint][3] and [pyflakes][4]; both of which are probably available as linters to your IDE.
- Don't include `__author__` in source code; commit history is used to keep track of who did what.
- An API is exposed either directly in `__init__.py`, or as a separate module called `api.py`. Prefer `__init__.py` unless you have reason not to. Don't forget that you must never remove anything from an API, so it's helpful to try and keep it as small as possible.
- Prefer relative imports, but don't get hung up about it if it slows you down.

This list will continue to grow and get more specific, watch this space.

[1]: https://www.python.org/dev/peps/pep-0008/
[2]: http://sphinxcontrib-napoleon.readthedocs.org/en/latest/example_google.html
[3]: http://www.pylint.org/
[4]: https://pypi.python.org/pypi/pyflakes