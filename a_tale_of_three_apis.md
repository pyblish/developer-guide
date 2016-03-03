## A Tale of Three APIs

In Pyblish, there are three conceptual APIs of interest while developing plug-ins.

- Library `->` Plug-in
- Plug-in `->` Library
- Plug-in `->` Plug-in

<br>
<br>
<br>

### Library to Plug-in

![image](https://cloud.githubusercontent.com/assets/2152766/7368404/6dcb2196-ed9f-11e4-9ce1-570697370d1f.png)

The Pyblish library itself carries an interface that is exposed to plug-ins.

This is the interface you gain access to from within your plug-in during development and includes helper functions exposed globally, such as [format_filename](http://api.pyblish.com/pages/format_filename.html).

```python
# Library API
>>> import pyblish.api
>>> pyblish.api.format_filename(r"str%*ange_fil£&en*ame.mb")
'strange_filename'
>>> pyblish.api.discover()
[..all discovered plug-ins..]
```

[ff]: http://api.pyblish.com/#format-filename

<br>
<br>
<br>

### Plug-in to Library

![image](https://cloud.githubusercontent.com/assets/2152766/7368408/75473d10-ed9f-11e4-8d41-230776dd7587.png)

Each plug-in also exposes attributes to the library. Primarily, attributes used to identifying the plug-in such its [family support](api.pyblish.com/pages/Plugin.families.html) and processing methods.

```python
import pyblish.api as pyblish

class ValidateInstance(pyblish.InstancePlugin):
    order = pyblish.ValidatorOrder # Run during validation

    families = ["myFamily"]  # Signals to the library that instances
                             # of family "myFamily" should be processed.
    hosts = ["maya"]  # This plug-in should run within Autodesk Maya

     def process(self, instance):
         # The library will call upon this method at the opportune time.
         instance.set_data("send_to_extraction", ["Some Data"])
```

This pattern is known as [Inversion-of-Control][ioc].

[ioc]: http://en.wikipedia.org/wiki/Inversion_of_control

<br>
<br>
<br>

### Plug-in to Plug-in

![image](https://cloud.githubusercontent.com/assets/2152766/7368412/7ccb0dfa-ed9f-11e4-8764-7612f05307f3.png)

Plug-ins may coordinate with each other by passing data.

```python
import pyblish.api as pyblish

class ValidateInstance(pyblish.InstancePlugin):
   order = pyblish.ValidatorOrder
 
   def process(self, instance):
       instance.set_data("message", "Hello")

class ExtractInstance(pyblish.InstancePlugin):
    order = pyblish.ExtractorOrder

    def process(self, instance):
        if "message" in instance.data:
            # ...
```

It does this via the [data member](http://api.pyblish.com/pages/AbstractEntity.data.html). Each stage, selection, validation, extraction and conform, may pass data from one to the next.

Other data includes switches within the instance itself. For example, a `review` family may include options for output resolution in pixels, or format such as whether to write a Quicktime file or a plain `png` sequence.

```python
instance.data["output"] = "png"
```