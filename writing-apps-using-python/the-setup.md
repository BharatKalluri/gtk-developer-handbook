---
description: Let's get started!
---

# The Setup

### Some Prerequisites

* Install `git`
* Sign up on a git hosting provider \([Github](https://github.com/)/[GitLab](https://gitlab.com/)/[GNOME's GitLab instance](https://gitlab.gnome.org/)\)

### Installing [GNOME Builder](https://wiki.gnome.org/Apps/Builder)

#### Install Builder using your distributions package manager

* Most of the distributions have Builder in their default repositories. You can directly install builder using your trusty package manager.

#### \(or\) Install Builder using Flatpak

* Install [Flatpak](https://www.flatpak.org/) \(Flatpak is a package/application manager\) , instructions to install flatpak on your distribution can be found [here](https://flatpak.org/setup/)
* Setup [flathub](https://flathub.org/) by issuing this command in the terminal

```text
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

* A restart is recommended at this point. Later issue this command in the terminal to install GNOME Builder. GNOME Builder is a [Integrated development environment \(IDE\)](https://en.wikipedia.org/wiki/Integrated_development_environment) made using GNOME technologies.

```text
flatpak install flathub org.gnome.Builder
```

That is about all you need to get started!

### What will we be building?

We will be building a desktop application which will fetch random pictures from [UnSplash](https://unsplash.com/) and set them as desktop wallpaper. 

The UI will be fairly simple, there will be one button on the header bar called shuffle. On press, it will fetch a new wallpaper, show the wallpaper on the window and automatically set the wallpaper. Our application will also have an about dialog and a shortcuts dialog.

### Creating a new project in Builder

Open Builder

![Builder start screen](../.gitbook/assets/image%20%281%29.png)

Click on `Start New Project` , Let us call our project name **Splash**! 

An App ID is a unique identifier for an application on the desktop. The idea is to make sure multiple applications in the same system do not conflict. This can be achieved by using something called the Reverse Domain Naming Notion \(or RDNN in short\). So let us name the App ID to be com.yourusername.splash \(replace your user name here, and make sure you do that step from here on\)

Select the language as Python and the we will leave the default license selection for now.

![Starting a new project ](../.gitbook/assets/image%20%282%29.png)

Looks good. Click on `Create Project` to create the project in your `Projects` folder!

Open the project, you will be welcomed with a screen like this

![GNOME Builder welcome screen](../.gitbook/assets/image%20%284%29.png)

Click on the first top left editor icon and click on `Build Preferences` 

![Select Build configuration screen](../.gitbook/assets/image%20%286%29.png)

Select the `Default` build configuration and click on `Make Active`. Now click on the Play \(▶️\) button to start running the application. There should be a new window which pops up saying Hello World!

![Hello World!](../.gitbook/assets/image%20%287%29.png)

Congratulations! Next up let us understand what are the files which were auto generated and how an application is actually built.

