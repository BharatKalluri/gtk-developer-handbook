---
description: Understanding what GNOME builder does to run your application
---

# The Build System

GNOME builder created a project using the [meson build system](https://mesonbuild.com/). 

Meson has a concept of using files which end with `.build` as the build configuration for that directory. Let us go through the default build configuration.

Open the `meson.build` at the root of the project. Here is what it contains

```text
project('splash',  
          version: '0.1.0',
    meson_version: '>= 0.50.0',
  default_options: [ 'warning_level=2',
                   ],
)

i18n = import('i18n')


subdir('data')
subdir('src')
subdir('po')

meson.add_install_script('build-aux/meson/postinstall.py')
```

* Line 1-5 : We are creating a new meson project with the name of the project as splash and the version to be 0.1.0.
* Line 8 : [i18n](https://www.wikiwand.com/en/Internationalization_and_localization) is used for internationalization. It will help us add translations for our desktop application.
* Line 11, 12, 13 : Adding sub directories `data` , `src`,`po`, this in turn triggers meson build inside those directories
* Line 15: Runs a python script at the end of the build

### Meson configuration in `data` folder

Let us now open the `meson.build` file inside `data` directory, these are the contents of the file

```text
desktop_file = i18n.merge_file(
  input: 'com.yourusername.splash.desktop.in',
  output: 'com.yourusername.splash.desktop',
  type: 'desktop',
  po_dir: '../po',
  install: true,
  install_dir: join_paths(get_option('datadir'), 'applications')
)

desktop_utils = find_program('desktop-file-validate', required: false)
if desktop_utils.found()
  test('Validate desktop file', desktop_utils,
    args: [desktop_file]
  )
endif

appstream_file = i18n.merge_file(
  input: 'com.yourusername.splash.appdata.xml.in',
  output: 'com.yourusername.splash.appdata.xml',
  po_dir: '../po',
  install: true,
  install_dir: join_paths(get_option('datadir'), 'appdata')
)

appstream_util = find_program('appstream-util', required: false)
if appstream_util.found()
  test('Validate appstream file', appstream_util,
    args: ['validate', appstream_file]
  )
endif

install_data('com.yourusername.splash.gschema.xml',
  install_dir: join_paths(get_option('datadir'), 'glib-2.0/schemas')
)

compile_schemas = find_program('glib-compile-schemas', required: false)
if compile_schemas.found()
  test('Validate schema file', compile_schemas,
    args: ['--strict', '--dry-run', meson.current_source_dir()]
  )
endif
```

#### Desktop file \(Line 1-8\)

i18n is responsible for internationalization. It takes `com.yourusername.splash.desktop.in` file as input and outputs the desktop file and installs it. Desktop files are responsible for the content you see on your launcher and task bar. Let us see what it contains. Open `com.yourusername.splash.desktop.in` 

```text
[Desktop Entry]
Name=splash
Exec=splash
Terminal=false
Type=Application
Categories=GTK;
StartupNotify=true
```

The desktop file currently contains the Name, Exec\(Executable to run\), Terminal\(Is it a terminal appplication?\), Type, Categories and StartupNotify. This file decides what text should be shown in the launcher etc.. You can read more about the spec [here](https://developer.gnome.org/desktop-entry-spec/). This file will be finally installed at `/usr/share/applications`.

the `get_option` function gets the appropriate folder path's for the input. Some common outputs for get option are as follows

```text
get_option('prefix') -> /usr
get_option('bindir') -> bin
get_option('datadir') -> /usr/share
```

> One concept prevalent here is the idea of `.in` files which are the input to a function. Which in turn generates files without the `.in` suffix and installs in a particular directory

#### Desktop file Validation \(Line 10-15\)

There is a program called `desktop-file-validate` which checks for errors in desktop files. Line 10 is trying to find the application. If it exists, then we use it to validate the generated desktop file and make sure there are no errors

#### AppStream \(Line 17-23\)

Appstream is a freedesktop specification which specifies metadata for applications. This is distro agnostic and a commonly agreed upon [spec](https://www.freedesktop.org/software/appstream/docs/). The appstream file is what is used to display information about the application in software centers. 

Appstream is stored in `.xml` format. The procedure is same as the desktop file. We take in the `.in`file and output the translated versions using i18n.

#### AppStream data validation \(Line 25-30\)

Again, a very similar piece of code to the desktop file validator. If the executable is found, meson runs the executable with the appstream file to make sure there are no errors.

#### GSchema file \(Line 32-34\)

`.gschema.xml` file is responsible for managing GSettings for your application. [GSettings](https://developer.gnome.org/GSettings/) are the settings you can define for your application. This is useful for persistant state and settings. For example, an application's window size and position can be stored in GSettings and retrieved whenever a new window is opened.

This file is installed in `/usr/share/glib-2.0/schemas`

#### GSchema validation \(Line 36-41\)

Similar to above validations. `glib-compile-schemas` compiles the `.gschema.xml` to make sure there are no errors.

We have covered what is in the `data/meson.build`. 

### Meson configuration in `po` directory

This directory contains the build file for the translations, we will look into it later.

### Meson configuration in `src` directory

Here are the contents of this file

```text
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

install_data(splash_sources, install_dir: moduledir)
```

Line1-3 : Imports and declaring some constants

#### GResource File \(Line 5-10\)

A Gresource file is responsible for listing out the resources required for your GTK Application. The file currently contains a pointer to one file as of now. 

```text
<?xml version="1.0" encoding="UTF-8"?>
<gresources>
  <gresource prefix="/com/yourusername/splash">
    <file>window.ui</file>
  </gresource>
</gresources>
```

Two important things to note, this file specifies a prefix under which files are stored and a list of files which will be stored.

#### Creating the executable \(Line 14-26\)

The python file `splash.py.in` is given as an input to a function which meson will process and return back a processed `splash` file which will be installed as an executable file in the bin directory. 

#### Storing the other source files \(Line 28-34\)

All the other python files which are used by the application are listed out in an array and then installed to the module directory.



That's it. Congratulations! With this knowledge it will be easier to understand how an application works. 

Next up, let us look into widgets!

