---
description: Let's get started!
---

# The Setup

### Some Prerequisites

* Install `git`
* Sign up on a git hosting provider \(Examples are Github, Gitlab etc..\)
* Install [Flatpak](https://www.flatpak.org/) , instructions to install flatpak on your distribution can be found [here](https://flatpak.org/setup/)
* Setup flathub by issuing this command in the terminal

  ```text
  flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
  ```

* A restart is recommended at this point. Later issue this command in the terminal to install GNOME Builder. GNOME Builder is a [Integrated development environment \(IDE\)](https://en.wikipedia.org/wiki/Integrated_development_environment) made using GNOME technologies.

```bash
flatpak install flathub org.gnome.Builder
```

That is about all you need to get started!

### What will we be building?

We will be building a desktop application which will fetch random pictures from [UnSplash](https://unsplash.com/) and set them as desktop wallpaper. 

The UI will be fairly simple, there will be one button on the header bar called shuffle. On press, it will fetch a new wallpaper, show the wallpaper on the window and automatically set the wallpaper. Our application will also have an about dialog and a shortcuts dialog. 

