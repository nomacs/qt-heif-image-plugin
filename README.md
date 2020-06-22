# qt-heif-image-plugin: Qt plugin for HEIF images
This is a Qt image plugin for [HEIF] images, wrapping [libheif]. It enables
existing Qt applications to open and save `.heic` files, such as those from
Apple devices.

[HEIF]: https://en.wikipedia.org/wiki/High_Efficiency_Image_File_Format
[libheif]: https://github.com/strukturag/libheif

Currently, support is limited to the following:
* Basic reading and writing of the primary image
* Reading of files with multiple top-level images

**Note:** This plugin is currently in progress for inclusion in
qtimageformats. Please see the Qt [Gerrit page] or [bug report] for updates.
Contributions should be made there, if possible. I will try to keep this
repository up to date.

[Gerrit page]: https://codereview.qt-project.org/236077
[bug report]: https://bugreports.qt.io/browse/QTBUG-69212

## Installation
### Ubuntu
A [PPA exists](https://launchpad.net/~jakar/+archive/ubuntu/qt-heif) for Ubuntu
users.  For *xenial* and earlier, the
[libheif repository](https://launchpad.net/~strukturag/+archive/ubuntu/libheif)
must also be enabled for dependencies.
```
$ sudo add-apt-repository ppa:jakar/qt-heif
$ sudo apt update
$ sudo apt install qt-heif-image-plugin
```

### Debian
Packages built against *buster* and *unstable* are provided on the
[latest release](https://github.com/jakar/qt-heif-image-plugin/releases/latest)
page.

### Fedora
Package is available in the RPM Fusion free repository.
To install it you need to enable RPM Fusion repo first - see [Enable RPM Fusion on your system](https://rpmfusion.org/Configuration).
Then it can be installed with following command:
```
$ sudo dnf install qt-heif-image-plugin
```

### Other systems
Users of other systems should build from source.

## Building from source
### Dependencies
Runtime dependencies:
- Qt 5 (Core and GUI modules)
- libheif (&ge; version 1.1)

Build-only dependencies:
- cmake
- pkg-config

### Build instructions
Get the source code:
```
$ git clone https://github.com/jakar/qt-heif-image-plugin.git
$ cd qt-heif-image-plugin
```

Configure with cmake and compile:
```
$ mkdir build
$ cd build
$ cmake ..
$ make
```

Finally, install:
```
$ sudo make install
```

The built library will be installed to `$QTDIR/plugins/imageformats`. Beware
that `CMAKE_INSTALL_PREFIX` will not be honored. Qt only searches in select
directories for plugins.

Alternatively, to use with a specific application, place the plugin in
`$APPDIR/imageformats`, where `$APPDIR` is the directory containing the
application's binary.

## Usage
Any application that (directly or indirectly) uses `QImageReader` to open image
files should automatically be able to use this plugin.

This has been successfully used with the following:
* [LXImage-Qt](https://github.com/lxqt/lximage-qt)
* [nomacs](https://github.com/nomacs/nomacs)

To test the plugin, [Dumageview](https://github.com/jakar/dumageview) was
created.
