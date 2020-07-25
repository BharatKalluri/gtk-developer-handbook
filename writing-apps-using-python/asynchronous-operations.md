---
description: Learning how to perform actions which do not block the main thread
---

# Asynchronous operations

We had `shuffle_image` in the `__init__` method. Which in turn caused the application to not load until the image was fetched from the API. Let us now make another function called async\_shuffle\_image which does not block on the main thread. 

We will be using the `threading` library to achieve this. Go through [this](https://realpython.com/intro-to-python-threading/#what-is-a-thread) article to understand how the threading library should be used. The gist of it is that, thread is a separate flow of execution. But these threads are not actual OS threads. So things will not technically be running in parallel. This is okay since our operations are not CPU Intensive but IO intensive. If they are CPU intensive, then `multiprocessing` module should be preferred.

A thread can be instantiated by using the `threading.Thread(fn, args)` method.This method will return a thread which will start running once we call the `.start()` method on it. You can also pickup a thread by using the `.join()` function.

A daemon is a process running in the background in CS terms. The threading.Thread function takes in a parameter called daemon which is a boolean. If it is true, then the thread will be pushed to the background. Now let us use this knowledge to make our shuffle function async.

```python
import threading

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
        self.async_shuffle_image()
        self.shuffle_button.connect("clicked", self.shuffle_button_on_clicked)
```

Let us walk through the code and note the changes which were done. 

* First up, we imported the `threading` module
* We also started using spinner here. At line 20, we started the spinner and at line 30 we are stopping the spinner. This indicates some IO operation is happening.
* At Line 32, we defined the `async_shuffle_image` function, which starts a daemon thread. At line 37, we changed the button to trigger the async function instead of the normal function.
* And finally at Line 42, we are calling the async function so that the startup time of the application is basically minimal but the image loads after a while.

Great! Now on startup, the application looks like this

![Loading screen](../.gitbook/assets/image%20%2824%29.png)

Also when we click on the `shuffle_button` the UI no longer freezes. Awesome!

