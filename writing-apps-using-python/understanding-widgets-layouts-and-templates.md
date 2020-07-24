---
description: Getting up to speed with GNOME templates
---

# Understanding Widgets, Layouts & Templates

Widgets are one of the most atomic components for GNOME. Everything you see on the screen in a GTK application is a widget. To create an application, we place a couple of widgets in a layout. 

`Button`, `Image`, `TextView`, `DrawingArea` are some examples of widgets. `Box` and `Grid` are some examples of Layouts.

Let us have a look at what `window.ui`, Right click on `window.ui` and click on `Open in` &gt; `UI desiginer`. This is what we have as of now

![The default hello world template](../.gitbook/assets/image%20%289%29.png)

The absolute parent component here is the `Application window` . Inside the application window, there is a `HeaderBar` component and a `label` component.  The `label` component has a ID of `label`. ID is important as it serves as a pointer later on while writing code. 

Let us have a look at what `window.py` contains

```text
from gi.repository import Gtk


@Gtk.Template(resource_path='/com/yourusername/splash/window.ui')
class SplashWindow(Gtk.ApplicationWindow):
    __gtype_name__ = 'SplashWindow'

    label = Gtk.Template.Child()

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```

The `SplashWindow` sub-classes `Gtk.ApplicationWindow` \(Since the absolute parent component in the template is a `Gtk.ApplicationWindow`\). The rule of thumb is that per template file, there will be one python class which sub-classes the absolute parent GTK type. The `__gtype_name__` of the class should be the same as the ID of the parent component.

`Gtk.Template.Child()` is used to retrieve the widget whose ID is the same as the variable name. In our case, the label widget's ID is `label`. So label will be a widget whose class will be `Gtk.Label`. 

This is the basic premise of templates in python.

### Customizing the UI layout

Now that we understand the basics, we won't be needing any layouts since the UI for the application is very simple. Let us go back to the designer view for `window.ui` and 

* Remove the `Gtk.Label` and replace the label widget with `Gtk.Image`. Set the ID of the component to `wallpaper_container`.
* Click on the headerbar and change the text from "Hello World" to "Splash". Make sure the ID of header bar is set to `header_bar` 
* In the options side bar when clicked on `header_bar` , change the last option \(`Number of Items`\) from 0 to 1. There will be an empty space which pops up to the left of the title of the header bar. Double click on the empty space and select `Gtk.Button` . 
* Set the ID of `Gtk.Button` to `shuffle_button` . Also set the text of the button to `Shuffle`

![The modified version of the template](../.gitbook/assets/image%20%2810%29.png)

Let us add a menu button, later we can add a pop over menu which will contain an `about` button. On click will show the about dialog.

Let's get back to editing window.ui . Click on the header bar and in the properties sidebar, increment Number of items from 1 to 2. You will see a new blank space pop up.

![Blank space for Menu Popover](../.gitbook/assets/image%20%2814%29.png)

Double click on empty space and add a Gtk.Button. Go to the properties sidebar of the Gtk.Button and set the ID of the button to menu\_button. Under the sub heading Button Content, Check on Add custom content.

![Blank area for Menu icon](../.gitbook/assets/image%20%2812%29.png)

Double click and add Gtk.Image. In the properties sidebar, Set the ID to open\_menu\_icon and Icon name to open-menu-symbolic. 

![Setting the menu icon on the button](../.gitbook/assets/image%20%2816%29.png)

Let us move the menu button to the other end. Having two buttons on the same side of the header bar is not good design. Click on the Menu button and go to the second tab in the properties sidebar \(named child container properties\). There is an option called Pack Type. This decides which side the widgets align. Think of the header bar as a Gtk.Box  

