https://github.com/pfalcon/pycopy/blob/master/README.md

> Pycopy - a minimalist and memory-efficient Python implementation. Good for desktop, cloud, constrained systems, microcontrollers, and just everything. - pfalcon/pycopy

# pfalcon/pycopy
[![Build Status](https://camo.githubusercontent.com/b946dcdf59c4c2eb399277caea3ff4350de5978f/68747470733a2f2f7472617669732d63692e6f72672f7066616c636f6e2f7079636f70792e706e673f6272616e63683d6d6173746572)](https://travis-ci.org/pfalcon/pycopy) [![Coverage Status](https://camo.githubusercontent.com/746b0764a6f1ac07c2267c64e2b3ea210e4fda24/68747470733a2f2f636f766572616c6c732e696f2f7265706f732f7066616c636f6e2f7079636f70792f62616467652e706e673f6272616e63683d6d6173746572)](https://coveralls.io/github/pfalcon/pycopy?branch=master)

Pycopy aims to develop and maintain a **minimalist**, lightweight, and extensible implementation of Python(-compatible) language. Pycopy to CPython is a similar thing as [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) to [Common Lisp](https://en.wikipedia.org/wiki/Common_Lisp). Pycopy works similarly well in the cloud, on desktop systems, on small embedded systems, and scales all the way down to microcontrollers. The project is developed and maintained by Paul Sokolovsky and is originally based on [MicroPython](http://www.micropython.org/), developed by Damien George, Paul Sokolovsky and contributors. Names "Pycopy" and "MicroPython" are used interchangeably in the project documentation and source code.

WARNING: this project is in beta stage and is subject to changes of the code-base, including project-wide name changes and API changes.

Pycopy implements the entire Python 3.4 syntax (including exceptions, `with`, `yield from`, etc., and additionally `async`/`await` keywords from Python 3.5). The following core datatypes are provided: `str` (including basic Unicode support), `bytes`, `bytearray`, `tuple`, `list`, `dict`, `set`, `frozenset`, `array.array`, `collections.namedtuple`, classes and instances. Builtin modules include `sys`, `time`, and `struct`, etc. Select ports have support for `_thread` module (multithreading). Note that only a subset of Python 3 functionality is implemented for the data types and modules.

Pycopy can execute scripts in textual source form or from precompiled bytecode, in both cases either from an on-device filesystem or "frozen" into the executable.

Pycopy is highly portable, and the main repository includes support for POSIX operating systems (Linux, MacOSX, FreeBSD, etc.), Windows, Android, and a number of bare-metal microcontroller systems (see below). Ports to other systems can be implemented easily. POSIX port (nicknamed "Unix port") is the reference port of Pycopy.

The Pycopy Zen
--------------

[Just as "big Python"](https://www.python.org/dev/peps/pep-0020/), Pycopy has its "Zen". The main principles of Pycopy are **simplicity**, **minimalism**, and **light-weightedness**.

At the same time, Pycopy strives to be a full-stack language and be compatible with wider Python ecosystem. The Pycopy project resolves these seemingly conflicting goals in a well-known and elegant way: by being a multi-level project, and by providing flexible configuration options. Specifically, there's a well-defined lightweight core written in C, defining the "native Pycopy language". Above it, a number of options are provided, implementing additional functionality (oftentimes offering more CPython compatibility). For example, on top of the core, "native Pycopy builtin modules" are provided, defining the native Pycopy API, which provides a subset of CPython's modules functionality, and at the same time, some extensions to it (driven by Pycopy's goal to be efficient). These native Pycopy modules are clearly namespaced, to allow to implement modules fully compatible with CPython API without any changes to the main project.

On top of this primary project, there are separate projects to further extend Pycopy functionality and achieve full-stack ecosystem. For example, there's a [pycopy-lib](https://github.com/pfalcon/pycopy-lib) project (see below) to implement a fully compatible CPython standard library for Pycopy.

Finally, on top of that infrastructure, there is an ecosystem of third-party packages, which are managed by the Pycopy users themselves.

The art of working with Pycopy is to understand where a particular feature belongs. Just as with CPython, it's almost never the core project, and almost always users' third party packages.

[Contributors' Guidelines](https://github.com/pfalcon/pycopy/wiki/ContributorGuidelines) further elaborate on some points touched above.

Source tree layout
------------------

Major components in this repository:

*   [py/](https://github.com/pfalcon/pycopy/blob/master/py) -- the core Python implementation, including compiler, runtime, and core library.
*   [mpy-cross/](https://github.com/pfalcon/pycopy/blob/master/mpy-cross) -- the bytecode (cross)compiler which is used to turn scripts into precompiled bytecode.
*   [ports/unix/](https://github.com/pfalcon/pycopy/blob/master/ports/unix) -- a version of Pycopy that runs on Unix (which includes Android).
*   [ports/windows/](https://github.com/pfalcon/pycopy/blob/master/ports/windows) -- a version for Windows.
*   [ports/stm32/](https://github.com/pfalcon/pycopy/blob/master/ports/stm32) -- a version of Pycopy that runs on the PyBoard and similar STM32 boards (using ST's Cube HAL drivers).
*   [ports/minimal/](https://github.com/pfalcon/pycopy/blob/master/ports/minimal) -- a minimal port. Start with this if you want to port the project to another microcontroller.
*   [tests/](https://github.com/pfalcon/pycopy/blob/master/tests) -- test framework and test scripts.
*   [docs/](https://github.com/pfalcon/pycopy/blob/master/docs) -- user documentation in Sphinx reStructuredText format. Rendered HTML documentation is available at [http://pycopy.readthedocs.io/](http://pycopy.readthedocs.io/) .

Additional components:

*   [ports/bare-arm/](https://github.com/pfalcon/pycopy/blob/master/ports/bare-arm) -- a bare minimum version for ARM MCUs. Used mostly to control code size.
*   [ports/teensy/](https://github.com/pfalcon/pycopy/blob/master/ports/teensy) -- a version that runs on the Teensy 3.1 (preliminary but functional).
*   [ports/pic16bit/](https://github.com/pfalcon/pycopy/blob/master/ports/pic16bit) -- a version for 16-bit PIC microcontrollers.
*   [ports/cc3200/](https://github.com/pfalcon/pycopy/blob/master/ports/cc3200) -- a version that runs on the CC3200 from TI.
*   [ports/esp8266/](https://github.com/pfalcon/pycopy/blob/master/ports/esp8266) -- a version that runs on Espressif's ESP8266 SoC.
*   [ports/esp32/](https://github.com/pfalcon/pycopy/blob/master/ports/esp32) -- a version that runs on Espressif's ESP32 SoC.
*   [ports/nrf/](https://github.com/pfalcon/pycopy/blob/master/ports/nrf) -- a version that runs on Nordic's nRF51 and nRF52 MCUs.
*   [extmod/](https://github.com/pfalcon/pycopy/blob/master/extmod) -- additional (non-core) modules implemented in C.
*   [tools/](https://github.com/pfalcon/pycopy/blob/master/tools) -- various tools, including the pyboard.py module.
*   [examples/](https://github.com/pfalcon/pycopy/blob/master/examples) -- various example scripts.

The subdirectories above may include READMEs with additional info.

"make" is used to build the components, or "gmake" on BSD-based systems. You will also need bash, gcc, and Python 3.3+ available as the command `python3` (if your system only has Python 2.7 then invoke make with the additional option `PYTHON=python2`).

The cross-compiler, mpy-cross
-----------------------------

Most ports require the Pycopy cross-compiler to be built first. This program, called mpy-cross, is used to pre-compile Python scripts to .mpy files which can then be included (frozen) into the firmware/executable for a port. To build mpy-cross use:

    $ cd mpy-cross
    $ make
    

The Unix version
----------------

The "unix" port requires a standard Unix environment with gcc and GNU make. x86 and x64 architectures are supported (i.e. x86 32- and 64-bit), as well as ARM and MIPS. Making full-featured port to another architecture requires writing some assembly code for the exception handling and garbage collection. Alternatively, fallback implementation based on setjmp/longjmp can be used.

To build (see section below for required dependencies):

    $ make -C mpy-cross
    $ cd ports/unix
    $ make submodules
    $ make
    

Then to give it a try:

    $ ./pycopy
    >>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
    

Use `CTRL-D` (i.e. EOF) to exit the shell. Learn about command-line options (in particular, how to increase heap size which may be needed for larger applications):

    $ ./pycopy --help
    

Run complete testsuite:

    $ make test
    

Unix version comes with a builtin package manager called `upip`, e.g.:

    $ ./pycopy -m upip install pycopy-pystone
    $ ./pycopy -m pystone
    

Browse available modules on [PyPI](https://pypi.org/search/?q=pycopy-). Standard library modules come from [pycopy-lib](https://github.com/pfalcon/pycopy-lib) project.

`pycopy` executable built following the instructions above is a "production" executable for native Pycopy software. It's also possible to build `pycop-dev` executable which provides additional reflection, diagnostics, and extensibility capabilities, at the expense of code size and memory usage efficiency. In particular, `pycopy-dev` is more compatible with software written for CPython. To build the `pycopy-dev` variant, run `make dev`.

External dependencies
---------------------

Building Pycopy ports may require some dependencies installed.

For Unix port, `libffi` library and `pkg-config` tool are required. On Debian/Ubuntu/Mint derivative Linux distros, install `build-essential` (includes toolchain and make), `libffi-dev`, and `pkg-config` packages.

Other dependencies can be built together with Pycopy. This may be required to enable extra features or capabilities, and in recent versions, these may be enabled by default. To build these additional dependencies, first fetch git submodules for them:

    $ make submodules
    

This will fetch all the relevant git submodules (sub repositories) that the port needs. Use the same command to get the latest versions of submodules as they are updated from time to time. After that execute:

    $ make deplibs
    

This will build all available dependencies (regardless whether they are used or not). If you intend to build Pycopy with additional options (like cross-compiling), the same set of options should be passed to `make deplibs`. To actually enable/disable use of dependencies, edit `ports/unix/mpconfigport.mk` file, which has inline descriptions of the options. For example, to build SSL module (required for `upip` tool described above, and so enabled by default), `MICROPY_PY_USSL` should be set to 1.

For some ports, building required dependences is transparent, and happens automatically. But they still need to be fetched with the `make submodules` command.

Contributing
------------

Pycopy is an open-source project and welcomes contributions which are aligned with its paradigm and work process. To be productive, please be sure to follow the [Contributors' Guidelines](https://github.com/pfalcon/pycopy/wiki/ContributorGuidelines) and the [Code Conventions](https://github.com/pfalcon/pycopy/blob/master/CODECONVENTIONS.md). Note that Pycopy is licenced under the MIT license, and all contributions should follow this license.

Q: Why fork?

A: I was a co-maintainer of MicroPython from 2013-12-29 till 2017-12-27, and authored 2779 commits during that time, that's more than 30% of commits at that timeframe ([prooflink](https://github.com/pfalcon/pycopy/graphs/contributors?from=2013-12-29&to=2017-12-27)). Many MicroPython's modules and subsystems were designed and implemented by me. After I lost commit access to the main repository, I continue my work here.

Q: Relation to the upstream?

A: I rebase on the upstream regularly. However, I revert/skip commits which are going in wrong direction with respect to Pycopy's goals (see "Zen" above), or not implemented thoroughly enough. I no longer contribute to upstream directly, due to upstream's failure to recognize me as a major contributor and uphold my copyright accordingly.

Q: Focus of the project?

A:

*   Code optimizations.
*   Continue to develop inplace, buffer and stream operations allowing to write highly memory effiicient applications.
*   Garbage collection experiments.
*   Reflection features (ultimately allowing to develop optimizing compilers, etc. in Python).
*   More CPython features implemented (configurable).
*   "Development/Testing" version with improved program analysis features.
*   etc.