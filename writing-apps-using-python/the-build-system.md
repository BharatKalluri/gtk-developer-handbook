---
description: Understanding what GNOME builder does to run your application
---

# The Build System

By default, GNOME builder created a project using the [meson build system](https://mesonbuild.com/). 

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

Let us now open the `meson.build` file inside `build` directory, these are the contents of the file

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

#### Desktop file

Line 1-8: i18n is responsible for internationalization. It takes `com.yourusername.splash.desktop.in` file as input and outputs the desktop file and installs it. Desktop files are resposible for the content you see on your launcher's and task bar's. Let us see what it contains. Open `data/com.yourusername.splash.desktop.in` 

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

the get\_option function gets the appropriate folder path's for the input. Some common outputs for get option are as follows

```text
get_option('prefix') -> /usr
get_option('bindir') -> bin
get_option('datadir') -> /usr/share
```

> One concept prevalent here is the idea of `.in` files which are the input to a function. Which in turn generates files without the `.in` suffix and installs in a particular directory



