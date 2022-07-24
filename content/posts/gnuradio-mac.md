---
title: "Building GNU Radio on macOS"
date: 2017-10-22T14:41:56+02:00
draft: false
---

Since this topic has been around for some time, I want to share my experience on how to successfully compile GNU Radio on macOS.

## Prequisities
* macOS 10.13
* MacPorts in `/opt/local`
* Other GNU Radio installation via `sudo port install gnuradio`

## Why build from souce?
This is a simple question. If you just want to use GNU Radio, go ahead and
install via MacPorts or Homebrew. The tricky part is to actually develop for
GNU Radio without the possibility of a source build.
[PyBOMBS](https://github.com/gnuradio/pybombs) is a great tool for developers
in order to install several GNU Radio environments simultaneously in different
prefixes. From there, different features/bugfixes can be implemented without
touching the global GNU Radio installation or braking anything.

## Tutorial
### 1. Get your Python installation straight
There is an issue around with PyBOMBS, which generally causes the problem that
Python dependencies don't get installed for the default interpreter, but for
the Python version specified in the recipes
([#423]((https://github.com/gnuradio/pybombs/issues/423))). If your default
interpeter is Python 2.7, there shouldn't be a problem. If another version is
set up, UHD most likely will fail to configure and you have to install all
dependencies by hand using the correct Python version. Find out your default
interpreter by
```
python --version
```
For me it's `3.5.2`, so I could use MacPorts to install dependencies like this
```
sudo macports install py35-mako
```
Another method is to force CMake to use your 2.7 interpreter, where
dependencies should be installed by PyBOMBS. For this, open your
`[prefix]/src/uhd/host/cmake/Modules/UHDPython.cmake` and change line 35 from
```
FIND_PACKAGE(PythonInterp)
```
to
```
FIND_PACKAGE(PythonInterp 2.7 REQUIRED)
```
With this, UHD should succeed to build.
### 2. Make CMake find Qt4
When trying to configure GNU Radio, most likely Qt4 won't be found. This is due
to a MacPorts [issue](https://trac.macports.org/ticket/49629) where `qmake` is
not installed in any `$PATH`. An easy workaround is to symlink `qmake` from
your Qt4 installation to `/opt/local/bin` by this command
```
sudo ln -s /opt/local/libexec/qt4/bin/qmake /opt/local/bin/qmake
```
After this, CMake should find Qt4 and configure correctly.

### 3. Fix linking against libgsl
When trying to build now, changes are high that linking will fail with `ld:
library not found for -lgsl`. I'm not 100% sure why this is happening, but most
likely clang behaves differntly than gcc here. A fix for this has been proposed
in [#1490](https://github.com/gnuradio/gnuradio/pull/1490). With this commit,
linking should succeed.

### 4. Make CMake use correct Python paths
This is the point where you have to `cd [prefix]/src/gnuradio/build` and invoke
`cmake` by hand, because some directories need to be passed manually. I guess
since there are several Python installations on macOS (MacPorts, system
default, ...), CMake gets mixed up and tries to spawn a thread in the wrong
interpreter, when `gnuradio-companion` is launched. To fix this, use the
`cmake` command like this and *replace [pybombs prefix] with your prefix*.
```
cmake .. -DCMAKE_INSTALL_PREFIX=[pybombs prefix]
-DPYTHON_EXECUTABLE=/opt/local/bin/python2.7
-DPYTHON_INCLUDE_DIR=/opt/local/Library/Frameworks/Python.framework/Versions/2.7/Headers
-DPYTHON_LIBRARY=/opt/local/Library/Frameworks/Python.framework/Versions/2.7/Python
-DSPHINX_EXECUTABLE=/opt/local/bin/rst2html-2.7.py
```
With this invokation, paths should be set up correctly. Go ahead and build &&
install with
```
make && make install
```
