# Working with GIO: GActions

[Actions](https://developer.gnome.org/gio/2.64/GAction.html) can be defined as a signals which can have attached callbacks. They are created to be a part of a group called action groups. GNOME by default creates two action groups for us, one for application and one for the window. The action group for application is called `app` and the action group for window is called `win`.

For example, we want to have an about dialog. An About dialog is something which is specific to the application itself and not to any specific window. There are multiple ways we can go about this. One way is that we can have a callback and on some buton click, we use the callback function to show up the about window. Or we can go the much cleaner way and setup an action called show\_about\_dialog on the application. And then call this GAction anywhere else in the application. 

One more very useful property of GAction is that we can assign a keyboard acclerator \(shortcuts\) very easily. 

#### About Dialog

The first step is to make the UI file for the about dialog. Let us create a file called about\_dialog.ui in the src directory. Open in UI designer and double click to add a about dialog.

![The default about dialog](../.gitbook/assets/image%20%2828%29.png)

You can fill in all the details you care about, I filled in the ID \(about\_dialog\), Name \(Splash\), Logo \(applications-graphics\), License \(GPL v3\) and Authors \(My name!\).

![Filling in some values for the about dialog](../.gitbook/assets/image%20%2829%29.png)

But this won't be included in the build unless we add this UI file in the gresource.xml file. So, quickly open the gresource.xml file and add it there. Here are the updated contents of splash.gresource.xml

```text
<?xml version="1.0" encoding="UTF-8"?>
<gresources>
  <gresource prefix="/com/yourusername/splash">
    <file>window.ui</file>
    <file>about_dialog.ui</file>
  </gresource>
</gresources>
```

Now let us jump back to `src/main.py` and write a function called `set_up_actions`. Before this, let us understand how to define a simple action. You can find the API documentation [here](https://lazka.github.io/pgi-docs/index.html#Gio-2.0/classes/SimpleAction.html#Gio.SimpleAction). We can write a helper function which will be creating a simple action, Connecting the action to a callback function, adding it to the application window \(remember application window already has a action group predefined\) and finally set up a keyboard accelerator if needed.

```python
import sys
import gi

gi.require_version('Gtk', '3.0')

from gi.repository import Gtk, Gio

from .window import SplashWindow


class Application(Gtk.Application):
    def __init__(self):
        super().__init__(application_id='com.yourusername.splash',
                         flags=Gio.ApplicationFlags.FLAGS_NONE)

    def set_up_actions(self):
        actions = [
            {
                'name': 'about',
                'func': self.show_about_dialog,
            },
        ]

        for a in actions:

            action_name: str = a['name']
            fn_for_action = a['func']
            accel_for_action = a.get('accel')

            c_action = Gio.SimpleAction.new(action_name, None)
            c_action.connect('activate', fn_for_action)
            self.add_action(c_action)
            if 'accel' in a.keys():
                self.set_accels_for_action(
                    f'app.{a["name"]}',
                    [accel_for_action],
                )

    def show_about_dialog(self, *args):
        print("Showing about dialog")
        builder = Gtk.Builder.new_from_resource('/com/yourusername/splash/about_dialog.ui')
        dialog = builder.get_object('about_dialog')
        dialog.set_modal(True)
        dialog.set_transient_for(self.props.active_window)
        dialog.present()


    def do_activate(self):
        win = self.props.active_window
        if not win:
            win = SplashWindow(application=self)
        self.set_up_actions()
        win.present()


def main(version):
    app = Application()
    return app.run(sys.argv)
```

Let's walk through the code which got added. from line 17-22. We are defining an array of actions we want to define. It will have a name, a function to execute and a optional accel. and later from line 24-36, we just create the action using `Gio.SimpleAction.new` and connect it with the function and setup the accel if needed.

From Line 39-45, we are getting a new `Gtk.Builder` instance from resource , getting the dialog and showing it. 

The next step is to make the menu button on the header bar functional. The menu button on press should come up with a popover menu. Which currently will have one option. `About`. On click should show the about dialog. GIO also gives us the convenience of creating a menu model based on actions. You can find the documentation for that [here](https://developer.gnome.org/gio/2.64/GMenuModel.html). 

```python
# src/window.py
import threading

from gi.repository import Gtk, GdkPixbuf, Gio
from .services.wallpaper_service import WallPaperService
from .services.unsplash_service import UnSplashService


@Gtk.Template(resource_path='/com/yourusername/splash/window.ui')
class SplashWindow(Gtk.ApplicationWindow):
    __gtype_name__ = 'SplashWindow'

    wallpaper_container: Gtk.Image = Gtk.Template.Child()
    shuffle_button: Gtk.Button = Gtk.Template.Child()
    loading_spinner: Gtk.Spinner = Gtk.Template.Child()
    open_menu_button: Gtk.MenuButton = Gtk.Template.Child()

    unsplash_service: UnSplashService = UnSplashService()
    wallpaper_service: WallPaperService = WallPaperService()

    def set_menu_items(self):
        menu: Gio.Menu = Gio.Menu()
        menu.append_item(Gio.MenuItem.new("About", "app.about"))
        self.open_menu_button.set_menu_model(menu)


    def shuffle_image(self):
        self.loading_spinner.start()
        random_wallpaper_url: str = self.unsplash_service.get_random_photo_url()
        self.wallpaper_service.set_wallpaper_from_url(random_wallpaper_url)
        pixbuf = GdkPixbuf.Pixbuf.new_from_file_at_scale(
            self.wallpaper_service.splash_wallpaper_file_path,
            width=1280,
            height=720,
            preserve_aspect_ratio=True,
        )
        self.wallpaper_container.set_from_pixbuf(pixbuf)
        self.loading_spinner.stop()

    def async_shuffle_image(self):
        thread = threading.Thread(target=self.shuffle_image, daemon=True)
        thread.start()

    def shuffle_button_on_clicked(self, button: Gtk.Button):
        self.async_shuffle_image()


    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.set_menu_items()
        self.async_shuffle_image()
        self.shuffle_button.connect("clicked", self.shuffle_button_on_clicked)
```

There are some very small additions here. First up at Line 16,  we also get the `open_menu_button` which is of type `Gtk.MenuButton`. And we have a helper function called `set_menu_items` which creates an instance of `Gio.Menu`, adds one menu item labeled About which inturn calls the `app.about` action. Also in Line 50, we call the set\_menu items function in the `__init__` method.

Run the application to test it now.

![Functional header bar menu](../.gitbook/assets/image%20%2827%29.png)

![The about dialog](../.gitbook/assets/image%20%2826%29.png)

GIO actions themselves are very powerful when used right. This is just one instance used to demonstrate how they can be helpful.

