---
description: 'Now that we have the templates set, let us add some functionality'
---

# Working with GIO: GSettings

Let us see what is the functionality we want our application to have

* On click of the shuffle button, the Gtk.Image widget should show a new random wallpaper from unsplash
* On image change, the application should also change the wallpaper of the desktop
* Pressing R should do the same action as pressing shuffle
* There should be an about dialog which tells you made the application
* There should be a shortcuts dialog which tells the user, R can be pressed to change the wallpaper

We will be covering the first two points in this chapter. The first step is to fetch random wallpapers from unsplash.

### Understanding GIO

GIO \(stands for GNOME Input/Output\) is a low level API for interacting with the GNOME stack. These are well defined API's which help us with

* File Operations
* File system monitoring
* Async IO
* Settings
* Permissions
* [and many more](https://developer.gnome.org/gio/2.26/)

So, most probably if you want to interact and work with the system you should be using GIO. In our case, we want to change the wallpaper of the system. Which is a setting we want to change. 

### What is GSettings?

When we mention we want to change GSettings, these are the settings variables which GNOME uses as the settings variables when running the desktop. 

This concept is not limited to just GNOME desktop. The application you are currently writing also can have settings. During [The Build System](the-build-system.md#gschema-file-line-32-34) chapter, when discussing about the gschema file. We did mention there will be settings like window height and width your application can store and retrieve. These settings can store a wide range of values and are extremely useful. To explore what are the GSettings you can tweak in your system right now, install [DConf Editor](https://flathub.org/apps/details/ca.desrt.dconf-editor) from flathub and have a look around.

![Desktop background settings in dconf application](../.gitbook/assets/image%20%2822%29.png)

If you venture into org/gnome/desktop/background and change the value of picture URI to any valid picture path you have on your desktop. The wallpaper on your desktop changes. Now, we need to figure out how to do it using python. 

The first step is to go to the API documentation for the pyobject libraries, [here](https://lazka.github.io/pgi-docs/index.html#Gio-2.0/classes/Settings.html#Gio.Settings) is the link to `Gio.Settings`. In the methods section, notice that there is a function called `set_string`. We can use this method to set the picture-uri property. To understand what is the schema name we need to modify the settings for,  click on the Picture URI and you can see the Schema mentioned there

![Explanation for Picture URI](../.gitbook/assets/image%20%2821%29.png)

#### Wallpaper service

With the new found knowledge, let us write a `wallpaper_service` which first downloads a file from a URL to a pre defined file path. And then another function which modifies the GSettings for the desktop background to change the wallpaper. 

Create a folder called `services`and create a file called `wallpaper_service.py` inside `services`.

```python
import os
from pathlib import Path
from gi.repository import Gio

import requests


class WallPaperService:
    splash_wallpaper_file_path = os.path.join(
        Path.home(),
        "Pictures",
        "splash_wallpaper.jpg"
    )
    background_settings = Gio.Settings.new("org.gnome.desktop.background")

    def set_wallpaper_from_file_uri(self, file_uri: str):
        self.background_settings['picture-uri'] = f"file:///{file_uri}"

    def write_image_url_to_wallpaper_file(self, image_url: str):
        response = requests.get(image_url, stream=True)
        if response.ok:
            with open(self.splash_wallpaper_file_path, "wb") as wallpaper_path:
                wallpaper_path.write(response.raw.read())

    def set_wallpaper_from_url(self, image_url: str):
        self.write_image_url_to_wallpaper_file(image_url)
        self.set_wallpaper_from_file_uri(self.splash_wallpaper_file_path)

```

Let us walk through the code, the class has a class variable called `splash_wallpaper_file_path` which is a constant path at which we will be saving the wallpaper. Another class variable called `background_settings` which holds GSettings for the schema which we got from exploring DConf \(`org.gnome.desktop.background`\).

The `set_wallpaper_from_file_uri` just takes in a URI and set's the GSetting accordingly. `set_wallpaper_from_file_uri` downloads an image from the URL and saves in a given path. The final function `set_wallpaper_from_url` combines both these functions, it takes in a `image_url`, downloads the file and places it in the desired file path and then changes the GSetting. Great!

#### UnSplash Service

Unsplash's API is fairly straightforward. Here is the [API Documentation for getting a random photo](https://unsplash.com/documentation#get-a-random-photo). You will be needing a `access key` for making a request which you can easily obtain by registering as a developer with Unsplash. Join us back once you have the API key!

Now that you have the API key, let us write a simple UnSplash Service which takes care of all the API interaction from Unsplash. In the `src` folder, create a file called `unsplash_service.py`

```python
# contents of src/services/unsplash_service.py

import requests


class UnSplashService:
    access_key = "<<your unsplash access key>>"

    def get_random_photo_url(self):
        print("Getting a random photo from UnSplash!")
        unsplash_response = requests.get(
            f"https://api.unsplash.com/photos/random",
            params={
                "client_id": self.access_key,
                "orientation": "landscape"
            }
        )
        response_json = unsplash_response.json()
        raw_url = response_json['urls']['raw']
        return raw_url

```

This is a fairly straight forward class. It stores the access key as a class variable and has a method which get's a random photo's URL using Unsplash's API. Note that we are using a new module called requests. If you do not have it installed, you can install by typing this into your terminal \(assuming you have pip already\)

```bash
pip install --user requests
```

Awesome! Let us now use these classes in our code so that clicking shuffle changes the wallpaper.



