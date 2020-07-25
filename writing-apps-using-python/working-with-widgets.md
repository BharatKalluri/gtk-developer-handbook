# Working with Widgets

We have all the helper services we need to start working on widgets now. window.py is currently outdated. There is one label pointer which no more exists in the template itself.

So, remove the `label` variable. And add a template child to `wallpaper_container`, `shuffle_button` and `loading_spinner`.

```python
# Contents of window.py
from gi.repository import Gtk


@Gtk.Template(resource_path='/com/yourusername/splash/window.ui')
class SplashWindow(Gtk.ApplicationWindow):
    __gtype_name__ = 'SplashWindow'

    wallpaper_container: Gtk.Image = Gtk.Template.Child()
    shuffle_button: Gtk.Button = Gtk.Template.Child()
    loading_spinner: Gtk.Spinner = Gtk.Template.Child()

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```

It is time to make the shuffle image work. Let us make a function called `shuffle_image` which gets a random wallpaper \(from `unsplash_service.py`\) and saves it to the disk in the designated path and then call the GSettings change function from wallpaper\(using `wallpaper_service.py`\)

#### Exploring what methods exist for Gtk classes

But how do we set the wallpaper\_container to show the image from the file? For that, we have to refer the [PyGObject API Documentation](https://lazka.github.io/pgi-docs/index.html). In that website, search for `Gtk.Image` and you will find yourself on [this page](https://lazka.github.io/pgi-docs/index.html#Gtk-3.0/classes/Image.html#Gtk.Image). Jump to the methods section and go through the method list to get an taste of what methods exist for `Gtk.Image`. You will notice that there is a method called [set\_from\_file](https://lazka.github.io/pgi-docs/index.html#Gtk-3.0/classes/Image.html#Gtk.Image.set_from_file). Since we have the complete path of the file, we can use that file path and use `set_from_file` to set the image container with the image. But set\_from\_file does not care about aspect ratio. But if we use set\_from\_pixbuf, and create a pixbuf, set it to preserve aspect ratio and can dictate the width and height of the Pixbuf. Then it will look better

Now, jump to `set_from_pixbuf` and click on PixBuf. You will be redirected [here](https://lazka.github.io/pgi-docs/index.html#GdkPixbuf-2.0/classes/Pixbuf.html#GdkPixbuf.Pixbuf). Jump to methods and you will find the method we want. We can create a new pixbuf using `new_from_file_at_scale` and specify all our parameters. And use that PixBuf to set the image in `wallpaper_container`.

> This is a process which you will find yourself doing repeatedly, currently the state of autocomplete is not very good\(although work is underway trying to make it better\). So to understand what methods exist for a Gtk/Gio/Glib class. You should jump to the API docs and explore.

Let us also get the shuffle button to shuffle image on click. Jump back into the documentation and look at the Gtk.Button docs. Switch to the [signals section](https://lazka.github.io/pgi-docs/index.html#Gtk-3.0/classes/Button.html#signals) and you can find a bunch of signals listed there. The one we need now is the clicked signal. Click on the clicked signal and [read the docs](https://lazka.github.io/pgi-docs/index.html#Gtk-3.0/classes/Button.html#Gtk.Button.signals.clicked). It takes in an argument of Gtk.Button.  

```python
# Contents of window.py
from gi.repository import Gtk, GdkPixbuf
from .services.wallpaper_service import WallPaperService
from .services.unsplash_service import UnSplashService


@Gtk.Template(resource_path='/com/yourusername/splash/window.ui')
class SplashWindow(Gtk.ApplicationWindow):
    __gtype_name__ = 'SplashWindow'

    wallpaper_container: Gtk.Image = Gtk.Template.Child()
    shuffle_button: Gtk.Button = Gtk.Template.Child()
    loading_spinner: Gtk.Spinner = Gtk.Template.Child()

    unsplash_service: UnSplashService = UnSplashService()
    wallpaper_service: WallPaperService = WallPaperService()

    def shuffle_image(self):
        random_wallpaper_url: str = self.unsplash_service.get_random_photo_url()
        self.wallpaper_service.set_wallpaper_from_url(random_wallpaper_url)
        pixbuf = GdkPixbuf.Pixbuf.new_from_file_at_scale(
            self.wallpaper_service.splash_wallpaper_file_path,
            width=1280,
            height=720,
            preserve_aspect_ratio=True,
        )
        self.wallpaper_container.set_from_pixbuf(pixbuf)
        
    def shuffle_button_on_clicked(self, button: Gtk.Button):
        self.shuffle_image()

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.shuffle_image()
        self.shuffle_button.connect("clicked", self.shuffle_button_on_clicked)
        
```

Let us recap on what we did here. We created a `shuffle_image` function which get's a `random_wallpaper_url`, set's the `wallpaper_from_url` \(which inturn also saves to disk and sets the Gsetting\) and later created a pixbuf from the file and set it to our `wallpaper_container`.

We also created a `shuffle_button_on_clicked` function which just calls back `shuffle_image` and connected the signal clicked from the button to trigger shuffle.

And we are calling the `shuffle_image`  function in the `__init__` method so that we get a random image on application load.

Try to run the application using builder and you will be greeted with an error. 

![Error saying services folder is not found](../.gitbook/assets/image%20%2823%29.png)

This makes sense, because we never included the services folder in the `src/meson.buid`. That is the reason this folder never was in the installed location. Let us quickly add this folder to the module so that the application can find it when installed using meson's [install\_subdir](https://mesonbuild.com/Reference-manual.html#install_subdir) method.

```text
# Contents of src/meson.build (Line 35 got added)
pkgdatadir = join_paths(get_option('prefix'), get_option('datadir'), meson.project_name())
moduledir = join_paths(pkgdatadir, 'splash')
gnome = import('gnome')

gnome.compile_resources('splash',
  'splash.gresource.xml',
  gresource_bundle: true,
  install: true,
  install_dir: pkgdatadir,
)

python = import('python')

conf = configuration_data()
conf.set('PYTHON', python.find_installation('python3').path())
conf.set('VERSION', meson.project_version())
conf.set('localedir', join_paths(get_option('prefix'), get_option('localedir')))
conf.set('pkgdatadir', pkgdatadir)

configure_file(
  input: 'splash.in',
  output: 'splash',
  configuration: conf,
  install: true,
  install_dir: get_option('bindir')
)

splash_sources = [
  '__init__.py',
  'main.py',
  'window.py',
]

install_subdir('services', install_dir: moduledir)
install_data(splash_sources, install_dir: moduledir)
```

We used install\_subdir function to install the services directory to module directory in line 35. Now run the application and it will run fine. After a while, you will see that the window loads with the image. And the same image will be in your Pictures folder and also be set as your desktop wallpaper!

But if you notice, the application has a huge startup time. That is because we are blocking on the main thread. The `__init__` method blocks on `shuffle_image` , loads the image and then let's the window rendering resume. This is bad user experience. Ideally we should load the image in the background and show a loading spinner until the image is loaded. Let us see how we can run background tasks using python next!

