# Integrating Blender in a Production Environment

Hey all. In the last year or so I had several discussions about the question:

`How do we integrate Blender in our Pipeline?`

While there are thousands of tutorials out there on how to do this or achieve that in Blender there are hardly any talking about how to use Blender in a Production Pipeline. But for Studios wanting to integrate Blender, this is almost the first question that needs an answer.

There is not really an official document somewhere that gives you an overview. So I though it's time to start a little repository to cover the frequently asked questions. Ideally this should be shared knowledge base that grows over time. I strongly encourage everyone to contribute to this document. The goal of this document should be to give TDs an overview on how to get started integrating Blender in a Pipeline. You will find useful links, tips and tools that make your life easier!

## Table of Contents
- [Starting Blender](#starting-blender)
    - [Command Line Arguments](#command-line-arguments)
    - [Environment Variables](#environment-variables)
- [Qt](#Qt)
- [Python](#python)
    - [Python Api](#python-api)
    - [Addons](#addons)
    - [Scripts](#scripts)
    - [Properties](#properties)
        - [ID Properties](#id-properties)
        - [Type Properties](#type-properties)
        - [Window Manager Properties](#window-manager-properties)
    <!-- - [Operators](#operators)
        - [UI](#operators-in-ui) -->
    - [UI](#ui)
    <!-- - [Handlers](#handlers)
    - [Third-party python modules](#third-party-python-modules) -->
- [Developer Tips](#development)
<!-- - [Data handling](#data-handling)
    - [Datablocks](#datablocks)
    - [Fake User](#fake-user) -->
<!-- - [IO](#io) -->
- [Community](#community)

<!-- "Auto Run Python Scripts"
"how to make sure python scripts that comes with blender files are executed eg"
"publishing and versioning what’s the best approach"
"workflows: animation caching yes or no"
"fake user"
"setting up library assets"
"actions and animations"
"overrides" (https://code.blender.org/2022/02/overrides-workshop/)
"blender libraries, relinking"
"BAT" -->

## Starting Blender

### Command Line Arguments

You can start Blender with several [Command Line Arguments](https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html).

Please check this link for a more detailed explanation of all the flags.

**Some useful examples:**

- Start Blender and execute a python script on startup
    ```
    blender --python /path/to/script.py
    ```

- Start Blender with list of add-ons enabled in addition to default add-ons
    ```
    blender --addons studio_pipeline,render_farm,blender_kitsu
    ```

- Start Blender allowing Python to use system environment variables such as `PYTHONPATH` (by default Blender ignores that variable).
    ```
    blender --python-use-system-env
    ```
- Start Blender but skip processing the Blender user config and scripts directory. Useful to debug if user configuration (addons, startup scripts) is source of error.
    ```
    blender --factory-startup
    ```

- Render .blend file in the background to relative output path, specifying frame counter and output format. Use whole frame range and add file extension to path.
    ```
    blender -b shot.blend -o //render/shot_###### -F OPEN_EXR_MULTILAYER  -x 1 -a
    ```

### Environment Variables

Blender has a very limited range of built int environment variables that control certain aspects.

You can find a list of the available environment variables at the bottom of this page:

[Environment Variables](https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html#environment-variables)

One of the most important environment variable you are going to use is:

`BLENDER_USER_SCRIPTS`

If you worked with other DCCs this variable is similar to `HOUDINI_PATH` or `MAYA_SCRIPT_PATH`.

`BLENDER_USER_SCRIPTS` is a directory location in which Blender searches for all kinds of files and sub folders to configure the Blender session that is starting up.

To learn more about what you can configure please refer to:

[Blender’s Directory Layout](https://docs.blender.org/manual/en/latest/advanced/blender_directory_layout.html)


If you have some sort of software starter at your Studio you can use this variable to supply Artists with some add-ons, start-up scripts and other things that your studio or project requires.

> **_NOTE:_** `BLENDER_USER_SCRIPTS` is a single path. Blender does not support a list of search paths yet like you might be used to from other DCCs. Many Pipelines do like to have a directory path for Studio wide tools and scripts and at least another one for only project specific stuff. This is not possible currently. You will need to find a way to work around it.

> **_NOTE:_** Setting `BLENDER_USER_SCRIPTS` results in Users not having their personal add-ons and scripts available that Blender normally loads from a sub path in the users home directory (See: [Blender’s Directory Layout](https://docs.blender.org/manual/en/latest/advanced/blender_directory_layout.html)). Be aware of that.

## Qt

Many Studios use [Qt](https://www.qt.io/) to built UIs for their tools.
As many DCCs actually use Qt themselves, integrating custom Interfaces with some Qt Python binding is usually quite straightforward.

Blender has it's own UI Framework (openGL and soon Vulkan & Metal backend) and therefore has no Qt Integration out of the box.

But it's still possible to run your Qt Interfaces without blocking Blender. Many people have solved this problem, please refer to the following links for some example solutions:

- [Blender Pyside2 Example](https://github.com/friedererdmann/blender_pyside2_example)
- [bqt](https://github.com/techartorg/bqt) (Starts Blender inside of a Qt Window)
- [Blender Shotgrid Integration](https://github.com/diegogarciahuerta/tk-blender/blob/d2c21fa53ab861886858388fbdc115e6d4e10a9d/resources/scripts/startup/Shotgun_menu.py#L90)
- [Blender Avalon Integration](https://gitlab.com/jasperges/avalon-core/-/blob/add-blender28-support/avalon/blender/ops.py)

It basically comes down to starting your own Qt Event Loop and process it's events in a modal operator.
To learn more about modal operators refer to this link:

[Modal Operators](https://docs.blender.org/api/current/bpy.types.Operator.html#modal-execution)

## Python

### Python-Api

Blender has a built in Python API. You can find the official documentation here:

[Blender Python API Documentation](https://docs.blender.org/api/current/index.html)

Make sure to read the introduction articles and gotchas on that page. They will get you started quickly.

A great video series to get started with scripting in Blender is [Scripting for Artists](https://www.youtube.com/watch?v=sOS2ID1ZN3A&list=PL1fkRtMmJ4OOrY20bOVlxn_PFYx9ly97j) by [Sybren Stüvel](https://stuvel.eu/).
### Addons

The most common way to expose your Python scripts and tools to Artists is through a Blender add-on. It's very easy to create a Blender add-on with Python that can be then enabled and disabled in the user's preferences.

Please refer to this guide here that walks you through the whole process of creating an add-on:

[Addon Tutorial](https://docs.blender.org/manual/en/latest/advanced/scripting/addon_tutorial.html)

You will notice that Blender offers a very powerful way with its add-on architecture for custom extension.

### Scripts

At any time you can create or open Python scripts in the Text Editor. From the Text Editor you can also run these scripts and quickly prototype that way.

Another useful feature are startup scripts. To add a script that runs on startup just place them in the blender configuration directory at this subpath:

`./scripts/startup/*.py`

Refer to [Blender’s Directory Layout](https://docs.blender.org/manual/en/latest/advanced/blender_directory_layout.html) if you are unsure where that is.


A not very well known feature is that you can enable "register" text data blocks. Which means they will get run when the blend file is loaded.

![image info](./res/images/register_script.jpg)

This is a technique often used by riggers to build UIs for their rig with Python.

To make sure that this text datablock comes with the rig on link/append, you can just create a reference to it in a custom property:

```python
rig.data['script'] = bpy.data.texts['myscript.py']
```

Speaking of properties, let's have a look at them in the next section.

### Properties

Sooner or later you will run in scenarios in which you deal with some custom data, be it some asset attributes that your pipeline requires or really any arbitray data that you want to save on something in your blend file.

In Blender this can be done through [Properties](https://docs.blender.org/api/current/bpy.props.html).

Custom properties can be added to any subclass of an `ID`, `Bone` and `PoseBone`.

How you create a custom property, depends on the scenario, which is why it can be a little bit confusing for beginners. As there is not a really a resource out there that explains properties in detail, this section will receive some extra attention.

#### ID Properties

Let's say you have the object `Suzanne` in your scene and you want to add some metadata on it. It as easy as doing this:

```python
bpy.data.objects["Suzanne"]["fruit"] = "Banana"
```

> **_NOTE:_** We use [] notation here to assign a value to the key 'fruit' as you would do with dictionaries


Notice that in the object properties panel you will find `fruit` showing up under the `Custom Properties` tab. So the command above is essentially the same as using the `bpy.ops.wm.properties_add()` operator that you can find at the top of the tab.


![image info](./res/images/property_str.jpg)

Checking out the type, returns `str` as expected.

```python
type(bpy.objects["Suzanne"]["fruit"])
>>> str
```

---

Let's try out some different data types.

```python
bpy.data.objects["Suzanne"]["favorite_fruits"] = ["Banana", "Coconut", "Mango"]
```

![image info](./res/images/property_list_str.jpg)

Let's see what `type()` returns:

```python
type(bpy.objects["Suzanne"]["favorite_fruits"])
>>> <list>
```
Notice if the data type was a list containing only strings Blender shows it as 'N Items' in the UI. But we can still edit the value if we press the gear icon:

![image info](./res/images/property_edit_list_str.jpg)

Als notice the type is declared as 'Python'.

---

Instead of using an array of strings let's try what happens when we use as an array of floats.


```python
bpy.data.objects["Suzanne"]["favorite_floats"] = [1.0, 2.0, 3.0]
```

![image info](./res/images/property_list_float.jpg)

Notice that Blender semms to be able to display a FloatArray integrated in the UI.

Let's check what `type()` returns:

```python
type(bpy.objects["Suzanne"]["favorite_numbers"])
>>> <class 'IDPropertyArray'>
```

Interesting! You might have expected to get `list` as we did before.

Also note another thing. If we click the gear icon we can now select a data `Subtype`. Let's select color. Blender now display the float array as a color gizmo.

![image info](./res/images/property_list_float_color.jpg)


It is **important** to understand that if the data type matches a certain structure, Blender converts it to a built in type that supports additional features (subtype, display in the UI, ...). If you want to know exactly what's going down skimm through the [source code](https://github.com/blender/blender/blob/master/source/blender/python/intern/bpy_rna.c#L7506).


---

Last but not least let's try what happens when we assign a dictionary as value:


```python
bpy.data.objects["Suzanne"]["shopping_list"] = {"Banana": 99, "Apples": 2, "Coconuts": 42}
```

Let's see what `type()` returns:

```python
type(bpy.objects["Suzanne"]["shopping_list"])
>>> <class 'IDPropertyGroup'>
```

Dictionaries are converted to the IDPropertyGroup type. You can still get the dictionary with:

```python
bpy.objects["Suzanne"]["shopping_list"].to_dict()
```

You can read more about the `IDPropertyGroup` and `IDPropertyArray` type [here](https://docs.blender.org/api/current/idprop.types.html#idprop.types.IDPropertyGroup).

---

#### Type Properties

But what if we want to have a property that is registered on all Objects. Maybe an attribute `for_export` that is either True or False and controls if this object should be exported by our custom studio exporter.

Rather than registering our property on a single object we will register it on the `Object` type like so:

```python
bpy.types.Object.for_export = False
```

Notice that we are using the dot notation here, as we are essentially adding a new class attribute to the class `Object`. Using brackets would throw an error here.

If we now select any Object in our scene and check the custom properties tab, you will notice nothing is there.

But if we still try to get that new property:

```python
bpy.data.objects["Suzanne"].for_export
>>> False
```

We get the default value we used when initializing it.

Let's set the property to something else than it's default value:

```python
bpy.data.objects["Suzanne"].for_export = True
>>> AttributeError: 'Object' object attribute 'for_export' is read-only
```

But we still want to be able to change that.

---

When we want to add new properties on a whole type we should use a class that is in the `bpy.props` module. You can find all supported data types [here](https://docs.blender.org/api/current/bpy.props.html).

Let's try that again with:

```python
bpy.types.Object.for_export = bpy.props.BoolProperty(default=False)
```
We can now also change the value of that property for one object:

```python
bpy.data.objects["Suzanne"].for_export = True
```

It is generally adviced to work with the properties in the `bpy.props` module because they can do a lot more really useful things!

For example we can define a subtype when we initialize the property, give it a name and a description that will be displayed when users hover it with their mouse.

```python
bpy.types.Object.export_path = bpy.props.StringProperty(
    name="Export Path",
    description="Output path when export script will be executed",
    subtype="FILE_PATH"
)

```

Because we specified `subtype="FILE_PATH"` if the property is displayed in the UI it will have a little browse file icon next to it and if you press it users can navigate to a file through the Blender File Browser. The property then gets the filepath of the selected file assigned.

That way we can also not change the property value to another type by accident:

```python
bpy.context.active_object.export_path = 2
>>> TypeError: bpy_struct: item.attr = val: Object.export_path expected a string type, not int
```

---

A very powerful feature is that we can also assign callback functions that are executed when the property is modified, when it gets written or read.

```python

def my_callback(self, context):
    print(f"Value got updated to: {self.export_path}")

bpy.types.Object.export_path = bpy.props.StringProperty(
    name="Export Path",
    description="Output path when export script will be executed",
    subtype="FILE_PATH",
    update=my_callback,
)
```

Try it out an see what happens.

---

> **_NOTE:_** When registering properties on types with the dot notation don't start to access or set these properties with brackets (object["export_path"]) after that. Otherwise this can get confusing for you and Blender.

>**_In a Pipeline_**: you would usually create those properties in the regisering section of your add-on. That way you can ensure that those properties exist and your add-on can work with them. Try to solve as much as possible with the built-in properties of the `bpy.props` module and register them on a whole type.

Check out the [props.py](https://developer.blender.org/diffusion/BSTS/browse/master/blender-kitsu/blender_kitsu/props.py) module of the blender-kitsu add-on to see an example how this can look.

Please refer to the [Property Definitions](https://docs.blender.org/api/current/bpy.props.html) section of the documentation if you want to learn more about this topic.

#### Annotated Properties

In Blender we can register a range of Blender type classes with:

```python
bpy.utils.register_class()
```
Some of those Blender type classes are:

```
bpy.types.Header
bpy.types.KeyingSetInfo
bpy.types.Menu
bpy.types.Operator
bpy.types.Panel
bpy.types.PropertyGroup
bpy.types.RenderEngine
bpy.types.UIList
```

Those classes can also contain custom properties. But the way we register custom properties on those classes works a little bit different.

Let's have a look at an example operator. Often you want to supply some input to an operator and execute logic depending on the input.

We can define a custom property on an Operator like so:

```python
import bpy
from typing import *

class my_OT_custom_operator(bpy.types.Operator):
    bl_idname = "my.custom_operator"
    bl_label = "Custom Operator"

    # Define custom property with an Annotation, using : instead of =
    filepath: bpy.props.StringProperty(name="File Path", subtype="FILE_PATH")

    def execute(self, context: bpy.types.Context) -> Set[str]:

        # Get filepath and do something
        filepath = self.filepath
        print(f"Doing something with this filepath: {self.filepath}")

        # Return
        return {"FINISHED"}

# Unregister if already registered
try:
    bpy.utils.unregister_class(my_OT_custom_operator)
except:
    pass

# Register
bpy.utils.register_class(my_OT_custom_operator)

```

And now we can do the following in a the interactive pyhton console:

```python
bpy.ops.my.custom_operator(filepath="/tmp/test.txt")
>>> Doing something with this filepath: /tmp/test.txt
{'FINISHED'}
```
Pretty cool!

So notice that we have to only annotate these properties as class attribtues (with types of `bpy.props). Internally when registering this class Blender actually checks the:

```
Class.__annotations__
```

attribute and grabs the annotated properties from there. When you try to initialize a class attribute like so:

```python
class my_OT_custom_operator(bpy.types.Operator):
    ...
    # = instead of :
    filepath = bpy.props.StringProperty(name="File Path", subtype="FILE_PATH")
```

You might run in to errors and the class won't register.

That you can actually pass a value for the `filepath` property of our operator as a key word argument in python is some special magic that is done by Blender when registering Operators.


> **_NOTE:_** Operators are a very powerful concept, you can learn more about them in the offical [Operator](https://docs.blender.org/api/current/bpy.types.Operator.html) section of the documentation.


---

But the annotation rule is the same for a ProperyGroup for example:

```python
import bpy

class MyPropertyGroup(bpy.types.PropertyGroup):

    # Annotate the properties we want
    my_float: bpy.props.FloatProperty(name="Some Floating Point")
    my_bool: bpy.props.BoolProperty(name="Toggle Option")
    my_string: bpy.props.StringProperty(name="String Value")

# First register our PropertyGroup class
bpy.utils.register_class(MyPropertyGroup)

# Then actually create a new property the scene type that 'points'
# to our PropertyGroup
bpy.types.Scene.my_settings = bpy.props.PointerProperty(type=MyPropertyGroup)

```
You can now access `my_float` like so:

```python
bpy.context.scene.my_settings.my_float
>>> 0.0
```
Property Groups are versy useful to as the name says 'group' multiple properties together. You can also nest them in to each other.

>**_In a Pipeline_**: you would usually register one property group on the required type and store all your properties inside that group to keep things nice and tidy!


#### Window Manager Properties

The [window manager](https://docs.blender.org/api/current/bpy.types.WindowManager.html) is a Blender data block that defines open windows and other user interface data. Because of it's nature that it is alwyas newly created for a Blender session it offers python sripts a greate place to store very temporary data that will be scrapped when the session ends.

To add a property to the window manager do:

```python
bpy.context.window_manager["temp_prop"] = 123
```

This property will be gone when you open a new blend file or reload the current one.

### UI

People completely new to Blender might not be aware of it so it's worth mentioning that the entire Blender UI is actually defined in Python. That means not only can you use Python to create you own UIs for Blender, you can even modify the existing one and fit it to the need of your Studio without touching any C code.

If you have enabled the `Developer Extras` option in the Preferences (Interface Tab) you can actually right click on any UI item and select `Edit Source`. This will open the python file in Blender the text editor and jump to the line that is responsible for that Element. Really useful!

## Developer Tips
If you develop tools for Blender make sure to enable `Developer Extras` and `Python Tooltips` in your Preferences.

![image info](./res/images/developer_extras.jpg)

---

You often want to adjust some properties when writing Python scripts for Blender. To find out how to actually change a property with python, right click on the property and select `Copy Full Data Path` which copies the data path of the property to the clipboard.


You can also do it the other way around.
Open a Info Editor and change any property or perform an action. You will see the action being printed out as a Python command in the Info Editor. Very useful!

---

If you are going to integrate Blender in your pipeline you will be writing your first add-on or script sooner or later. In order to make your life easier consider using the following tools:

- [fake-bpy-module](https://github.com/nutti/fake-bpy-module): Is a python package that mimics the Blender Python API so you can have code completion in your IDE. Simply install it with: `pip install fake-bpy-module-latest
`

- [Blender Development VSCode Extension](https://marketplace.visualstudio.com/items?itemName=JacquesLucke.blender-development): Really useful extension for Visual Studio Code to make Blender development easier. One of the most important aspect is that it attaches a debugger and you can set breakpoints in your Python code.

## Community

Blender has a huge community and you can find help in a lot of places. The most important pages are:

- [developer.blender.org](https://developer.blender.org/): Phabricator front end that actually tracks the whole Blender development. Here you can find all open tasks, planned features, modules and their projects and open bug reports. This is the place to check out if you want to know in real time what's being worked on. **_NOTE:_** Will soon move to [Gitea](https://code.blender.org/2022/07/gitea-diaries-part-1/)

- [devtalk.blender.org](https://devtalk.blender.org/): This is a dedicated place for Blender module teams to reach out to contributors and the community. It's a good place to start discussions and ask questions in dedicated thread. You will also find reports of regular meetings amongst the developers. One of the most exciting ones to check out is the [weekly report](https://devtalk.blender.org/c/blender/weekly-updates) what changed in Blender.

- [blender.chat](https://blender.chat/): One of the most important pages to hang around and get help quickly. This is the official chat platform. It has dedicated [channels](https://blender.chat/directory/channels) for each module. You can think about this like your Studio Slack channel except everything is open. Even all the developers communicate publicly there, it's a great place to be.

- [righclickselect](https://blender.community/c/rightclickselect): This is the place to make feature requests for Blender. Share your idea with the community. Discuss it. Revise it.
