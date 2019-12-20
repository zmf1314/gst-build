# gst-build

GStreamer [meson](http://mesonbuild.com/) based repositories aggregrator.

Check out this module and run meson on it, and it will git clone the other
GStreamer modules as [meson subprojects](http://mesonbuild.com/Subprojects.html)
and build everything in one go. Once that is done you can switch into an
development environment which allows you to easily develop and test the latest
version of GStreamer without the need to install anything or touch an existing
GStreamer system installation.

## Getting started

### Install git and python 3.5+

If you're on Linux, you probably already have these. On macOS, you can use the
[official Python installer](https://www.python.org/downloads/mac-osx/).

You can find [instructions for Windows below](#windows-prerequisites-setup).

### Install meson and ninja

Meson 0.48 or newer is required.

On Linux and macOS you can get meson through your package manager or using:

  $ pip3 install --user meson

This will install meson into `~/.local/bin` which may or may not be included
automatically in your PATH by default.

You should get `ninja` using your package manager or download the [official
release](https://github.com/ninja-build/ninja/releases) and put the `ninja`
binary in your PATH.

You can find [instructions for Windows below](#windows-prerequisites-setup).

### Build GStreamer and its modules

You can get all GStreamer built running:

```
meson build/
ninja -C build/
```

This will automatically create the `build` directory and build everything
inside it.

NOTE: On Windows, you *must* run this from inside the Visual Studio command
prompt of the appropriate architecture and version.

# Development environment

## Building the Qt5 QML plugin

If `qmake` is not in `PATH` and pkgconfig files are not available, you can
point the `QMAKE` env var to the Qt5 installation of your choosing before
running `meson` as shown above.

The plugin will be automatically enabled if possible, but you can ensure that
it is built by passing `-Dgst-plugins-good:qt5=enabled` to `meson`. This will
cause Meson to error out if the plugin could not be enabled. This also works
for all plugins in all GStreamer repositories.

## Development environment target

gst-build also contains a special `devenv` target that lets you enter an
development environment where you will be able to work on GStreamer
easily. You can get into that environment running:

```
ninja -C build/ devenv
```

If your operating system handles symlinks, built modules source code will be
available at the root of `gst-build/` for example GStreamer core will be in
`gstreamer/`. Otherwise they will be present in `subprojects/`. You can simply
hack in there and to rebuild you just need to rerun `ninja -C build/`.

NOTE: In the development environment, a fully usable prefix is also configured
in `gst-build/prefix` where you can install any extra dependency/project.

An external script can be run in development environment with:

```
./gst-env.py external_script.sh
```

## Update git subprojects

We added a special `update` target to update subprojects (it uses `git pull
--rebase` meaning you should always make sure the branches you work on are
following the right upstream branch, you can set it with `git branch
--set-upstream-to origin/master` if you are working on `gst-build` master
branch).

Update all GStreamer modules and rebuild:

```
ninja -C build/ update
```

Update all GStreamer modules without rebuilding:

```
ninja -C build/ git-update
```

## Custom subprojects

We also added a meson option, `custom_subprojects`, that allows the user
to provide a comma-separated list of subprojects that should be built
alongside the default ones.

To use it:

```
cd subprojects
git clone my_subproject
cd ../build
rm -rf * && meson .. -Dcustom_subprojects=my_subproject
ninja
```

## Run tests

You can easily run the test of all the components:

```
meson test -C build
```

To list all available tests:

```
meson test -C build --list
```

To run all the tests of a specific component:

```
meson test -C build --suite gst-plugins-base
```

Or to run a specific test file:

```
meson test -C build/ --suite gstreamer gst_gstbuffer
```

Run a specific test from a specific test file:

```
GST_CHECKS=test_subbuffer meson test -C build/ --suite gstreamer gst_gstbuffer
```

## Optional Installation

`gst-build` has been created primarily for [development usage](#development-environment-target),
but you can also install everything that is built into a predetermined prefix like so:

```
meson --prefix=/path/to/install/prefix build/
ninja -C build/
meson install -C build/
```

Note that the installed files have `RPATH` stripped, so you will need to set
`LD_LIBRARY_PATH`, `DYLD_LIBRARY_PATH`, or `PATH` as appropriate for your
platform for things to work.

## Checkout another branch using worktrees

If you need to have several versions of GStreamer coexisting (eg. `master` and `1.14`),
you can use the `checkout-branch-worktree` script provided by `gst-build`. It allows you
to create a new `gst-build` environment with new checkout of all the GStreamer modules as
[git worktrees](https://git-scm.com/docs/git-worktree).

For example to get a fresh checkout of `gst-1.14` from a `gst-build` in master **already
built** in a `build` directory you can simply run:

```
./checkout-branch-worktree ../gst-build-1.14 origin/1.14 -C build/
```

This will create a new ``gst-build-1.14`` folder at the same level of ``gst-build`` pointing to the given branch ie *1.14*
for all the subprojects ( gstreamer, gst-plugins-base etc.)


## Add information about GStreamer development environment in your prompt line

### Bash prompt

We automatically handle `bash` and set `$PS1` accordingly.

If the automatic `$PS1` override is not desired (maybe you have a fancy custom prompt), set the `$GST_BUILD_DISABLE_PS1_OVERRIDE` environment variable to `TRUE` and use `$GST_ENV` when setting the custom prompt, for example with a snippet like the following:

```bash
...
if [[ -n "${GST_ENV-}" ]];
then
  PS1+="[ ${GST_ENV} ]"
fi
...

```

### Zsh prompt

In your `.zshrc`, you should add something like:

```
export PROMPT="$GST_ENV-$PROMPT"
```

### Using powerline

In your powerline theme configuration file (by default in
`{POWERLINE INSTALLATION DIR}/config_files/themes/shell/default.json`)
you should add a new environment segment as follow:

```
{
  "function": "powerline.segments.common.env.environment",
  "args": { "variable": "GST_ENV" },
  "priority": 50
},
```

## Windows Prerequisites Setup

On Windows, some of the components may require special care.

### Git for Windows

Use the [Git for Windows](https://gitforwindows.org/) installer. It will
install a `bash` prompt with basic shell utils and up-to-date git binaries.

During installation, when prompted about `PATH`, you should select the
following option:

![Select "Git from the command line and also from 3rd-party software"](/data/images/git-installer-PATH.png)

### Python 3.5+ on Windows

Use the [official Python installer](https://www.python.org/downloads/windows/).
You must ensure that Python is installed into `PATH`:

![Enable Add Python to PATH, then click Customize Installation](/data/images/py-installer-page1.png)

You may also want to customize the installation and install it into
a system-wide location such as `C:\PythonXY`, but this is not required.

### Ninja on Windows

The easiest way to install Ninja on Windows is with `pip3`, which will download
the compiled binary and place it into the `Scripts` directory inside your
Python installation:

```
pip3 install ninja
```

You can also download the [official release](https://github.com/ninja-build/ninja/releases)
and place it into `PATH`.

### Meson on Windows

**IMPORTANT**: Do not use the Meson MSI installer since it is experimental and known to not
work with `gst-build`.

You can use `pip3` to install Meson, same as Ninja above:

```
pip3 install meson
```

Note that Meson is written entirely in Python, so you can also run it as-is
from the [git repository](https://github.com/mesonbuild/meson/) if you want to
use the latest master branch for some reason.


### Setup a mingw/wine based development environment on linux

#### Install wine and mingw

##### On fedora x64

``` sh
sudo dnf install mingw64-gcc mingw64-gcc-c++ mingw64-pkg-config mingw64-winpthreads wine
```

FIXME: Figure out what needs to be installed on other distros

#### Get meson from git

This simplifies the process and allows us to use the cross files
defined in meson itself.

``` sh
git clone https://github.com/mesonbuild/meson.git
```

#### Build and install

```
BUILDDIR=$PWD/winebuild/
export WINEPREFIX=$BUILDDIR/wine-prefix/ && mkdir -p $WINEPREFIX
# Setting the prefix is mandatory as it is used to setup symlinks during uninstalled development
meson/meson.py $BUILDDIR --cross-file meson/cross/linux-mingw-w64-64bit.txt -Dgst-plugins-bad:vulkan=disabled -Dorc:gtk_doc=disabled --prefix=$BUILDDIR/wininstall/ -Djson-glib:gtk_doc=disabled
meson/meson.py install -C $BUILDDIR/
```

> __NOTE__: You should use `meson install -C $BUILDDIR`  each time you make a change
> instead of the usual `ninja -C build` as the environment is not uninstalled.

#### The development environment

You can get into the development environment the usual way:

```
ninja -C $BUILDDIR/ devenv
```

After setting up [binfmt] to use wine for windows binaries,
you can run GStreamer tools under wine by running:

```
gst-launch-1.0.exe videotestsrc ! glimagesink
```

[binfmt]: http://man7.org/linux/man-pages/man5/binfmt.d.5.html
