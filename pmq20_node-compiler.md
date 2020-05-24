pmq20/node-compiler

###    README.md

# [(L)](https://github.com/pmq20/node-compiler#nodejs-compiler)Node.js Compiler

Compiling your Node.js application into a single executable.

[![nodec.gif](../_resources/d84c571e161269340dd93d4c8c5d6424.gif)](https://github.com/pmq20/node-compiler/raw/master/nodec.gif)

## [(L)](https://github.com/pmq20/node-compiler#download)Download

|     | Master CI | [RAM Test](https://github.com/pmq20/node-compiler-ram) | [Black‑box Test](https://github.com/pmq20/node-compiler-blbt) | Latest Stable |
| --- | --- | --- | --- | --- |
| **Windows** | [[status](../_resources/905b560a6c3db4b662f364c1f45ac6cf.bin)](https://ci.appveyor.com/project/pmq20/node-compiler/branch/master) | [[status](../_resources/19dd3b51b968d01daad1ab1436552192.bin)](https://ci.appveyor.com/project/pmq20/node-compiler-ram/branch/master) | [[status](../_resources/905b560a6c3db4b662f364c1f45ac6cf.bin)](https://ci.appveyor.com/project/pmq20/node-compiler-blbt) | https://sourceforge.net/projects/node-compiler/files/v0.9.5/nodec.exe/download |
| **macOS** | [[status](../_resources/af3359b6a330e8be6d21e0274c77bc30.bin)](https://travis-ci.org/pmq20/node-compiler) | [(L)](https://travis-ci.org/pmq20/node-compiler-ram) | [(L)](https://travis-ci.org/pmq20/node-compiler-blbt) | https://sourceforge.net/projects/node-compiler/files/v0.9.5/nodec-darwin-x64/download |
| **Linux** | [(L)](https://travis-ci.org/pmq20/node-compiler) | [(L)](https://travis-ci.org/pmq20/node-compiler-ram) | [(L)](https://travis-ci.org/pmq20/node-compiler-blbt) | https://sourceforge.net/projects/node-compiler/files/v0.9.5/nodec-linux-x64/download |

## [(L)](https://github.com/pmq20/node-compiler#how-it-works)How it works?

### [(L)](https://github.com/pmq20/node-compiler#presentation)Presentation

[Node.js Compiler: compiling your Node.js application into a single executable](https://speakerdeck.com/pmq20/node-dot-js-compiler-compiling-your-node-dot-js-application-into-a-single-executable).

### [(L)](https://github.com/pmq20/node-compiler#comparing-with-similar-projects)Comparing with Similar Projects

Project
Differences
[pkg](https://github.com/zeit/pkg)

Pkg hacked [object Object] API's dynamically in order to access in-package files, whereas Node.js Compiler leaves them alone and instead works on a deeper level via [libsquash](https://github.com/pmq20/libsquash). Pkg uses JSON to store in-package files while Node.js Compiler uses the more sophisticated and widely used SquashFS as its data structure.

[EncloseJS](http://enclosejs.com/)

EncloseJS restricts access to in-package files to only five [object Object] API's, whereas Node.js Compiler supports all [object Object] API's. EncloseJS is proprietary licensed and charges money when used while Node.js Compiler is MIT-licensed and users are both free to use it and free to modify it.

[Nexe](https://github.com/nexe/nexe)

Nexe does not support dynamic [object Object] because of its use of [object Object], whereas Node.js Compiler supports all kinds of [object Object] including [object Object].

[asar](https://github.com/electron/asar)

Asar keeps the code archive and the executable separate while Node.js Compiler links all JavaScript source code together with the Node.js virtual machine and generates a single executable as the final product. Asar uses JSON to store files' information while Node.js Compiler uses SquashFS.

[AppImage](http://appimage.org/)

AppImage supports only Linux with a kernel that supports SquashFS, while Node.js Compiler supports all three platforms of Linux, macOS and Windows, meanwhile without any special feature requirements from the kernel.

## [(L)](https://github.com/pmq20/node-compiler#install)Install

### [(L)](https://github.com/pmq20/node-compiler#windows)Windows

First install the prerequisites:

- [SquashFS Tools 4.3](https://github.com/pmq20/squashfuse/files/691217/sqfs43-win32.zip)
- [Python 2.6 or 2.7](https://www.python.org/downloads/)
- [Visual Studio 2015 Update 3](https://www.visualstudio.com/), all editions including the Community edition (remember to select "Common Tools for Visual C++ 2015" feature during installation).

Then download the executable [nodec.exe](https://sourceforge.net/projects/node-compiler/files/v0.9.5/nodec.exe/download) and run it from the VC++ or VS Command Prompt.

### [(L)](https://github.com/pmq20/node-compiler#macos)macOS

First install the prerequisites:

- [SquashFS Tools 4.3](http://squashfs.sourceforge.net/): ` brew install squashfs `
- [Xcode](https://developer.apple.com/xcode/download/)
    - You also need to install the ` Command Line Tools ` via Xcode. You can find this under the menu ` Xcode -> Preferences -> Downloads `
    - This step will install ` gcc ` and the related toolchain containing ` make `
- Python 2.6 or 2.7
- GNU Make 3.81 or newer

Then,

	curl -L https://sourceforge.net/projects/node-compiler/files/v0.9.5/nodec-darwin-x64/download > nodec
	chmod +x nodec
	./nodec

### [(L)](https://github.com/pmq20/node-compiler#linux)Linux

First install the prerequisites:

- [SquashFS Tools 4.3](http://squashfs.sourceforge.net/)
- ` gcc ` and ` g++ ` 4.8.5 or newer, or
- ` clang ` and ` clang++ ` 3.4 or newer
- Python 2.6 or 2.7
- GNU Make 3.81 or newer

Then,

	curl -L https://sourceforge.net/projects/node-compiler/files/v0.9.5/nodec-linux-x64/download > nodec
	chmod +x nodec
	./nodec

## [(L)](https://github.com/pmq20/node-compiler#usage)Usage

	nodec [OPTION]... ENTRANCE
	  -r, --root=DIR                   Specifies the path to the root of the application
	  -o, --output=FILE                Specifies the path of the output file
	  -d, --tmpdir=DIR                 Specifies the directory for temporary files
	      --clean-tmpdir               Cleans all temporary files that were generated last time
	      --keep-tmpdir                Keeps all temporary files that were generated last time
	      --make-args=ARGS             Passes extra arguments to make
	      --vcbuild-args=ARGS          Passes extra arguments to vcbuild.bat
	  -n, --npm=FILE                   Specifies the path of npm
	      --npm-package=NAME           Downloads and compiles the specified npm package
	      --npm-package-version=VER    Downloads and compiles the specified version of the npm package
	      --debug                      Enable debug mode
	  -v, --version                    Prints the version of nodec and exit
	      --node-version               Prints the version of the Node.js runtime and exit
	  -h, --help                       Prints this help and exit

## [(L)](https://github.com/pmq20/node-compiler#example)Example

	git clone --depth 1 https://github.com/jashkenas/coffeescript.git
	cd coffeescript
	nodec bin/coffee
	./a.out (or a.exe on Windows)

## [(L)](https://github.com/pmq20/node-compiler#see-also)See Also

- [SquashFS](http://squashfs.sourceforge.net/): a compressed read-only filesystem for Linux.
- [Libsquash](https://github.com/pmq20/libsquash): portable, user-land SquashFS that can be easily linked and embedded within your application.
- [Enclose.IO](http://enclose.io/): cloud-based service that compiles your application into a single executable.