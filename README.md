# C/C++ Makefile Template #

[version_shield]: https://img.shields.io/badge/version-2.3.0-blue.svg
[latest_release]: https://github.com/mfederczuk/makefile-template/releases/latest "Latest Release"
[![version: 2.3.0][version_shield]][latest_release]
[![Changelog](https://img.shields.io/badge/-Changelog-blue)](./CHANGELOG.md "Changelog")

## About ##

**C/C++** [Makefile](https://www.gnu.org/software/make/) template for
 applications and libraries.

Writing a good Makefile is tedious and takes time. Time, that you could've spent
 writing your software.  
Not to mention, most Makefiles are build up in a way, where you need to list all
 of the object files that you want to create by hand.  
You then spend minutes trying to figure out why your program is not compiling
 just to figure out that it's never even building that file that you wrote an
 hour ago.

After an initial configuration, the Makefile will automatically detect any
 source files in a specified directory and it's subdirectories.  
Building object files, building executables or shared and static libraries,
 building test executables, installing, uninstalling and cleaning is all covered
 by this template.

## Download ##

Download the Makefile from the [latest release][latest_release] and move it into
 your project directory.

You can also use `wget` to quickly get it from my **GitHub Pages** site.

	wget https://mfederczuk.github.io/makefile-template/Makefile

## Configuration ##

After you've added the Makefile into your project directory, you will need to
 configure it a little.  
Open it up and you will see various variable definitions under the
 "user definitions" section.

* `SOFTWARE`  
  The Makefile lets you build two kinds of software: executables and libraries.  
  Set this variable to `exe` to build an executable, set it to `lib` to build
   a shared (dynamic) and static library.  
  Any other value is invalid and will result in errors when trying to invoke the
   Makefile
* `TARGET`  
  The name of the executable or library to build.  
  Don't add any prefix or suffixes to this name. e.g.: instead of "`libfoo.so`"
   use just "`foo`".  
  If you want to change any prefix or suffixes you can do so by changing the
   `exe_prefix`, `exe_suffix`, `shared_lib_prefix`, `shared_lib_suffix`,
   `static_lib_prefix` and `static_lib_suffix` variables.
* `PACKAGE`  
  The name of the package the target belongs to.
* `SRC_MAIN`  
  The directory in which the source files are stored.  
  The Makefile will search this directory for any source files and will
   automatically add them.  
  Any file that is not a **C** or **C++** file will be ignored.  
  To see what file extensions are mapped to which language, refer to
   [Appendix A](#appendix-a-file-extensions)
* `SRC_TEST`  
  The directory in which the test source files are stored.  
  Each source file corresponds to one test file.  
  The Makefile will search this directory for any source files and will
   automatically add them.  
  Any file that is not a **C** or **C++** file will be ignored.  
  To see what file extensions are mapped to which language, refer to
   [Appendix A](#appendix-a-file-extensions)
* `SRC`  
  The same as `SRC_MAIN` but without tests enabled.  
  You must choose between using either only `SRC` or both `SRC_MAIN` and
   `SRC_TEST`
* `BIN`  
  The directory in which the built object files will be stored.  
  It's generally a good idea to add this directory to your `.gitignore` file.  
  This directory does not need to exist before building.
* `INC`  
  The directory in which the header files are stored.  
  It's not needed to define this variable if you're building an executable,
   since we only need to know this directory so we can install it.  
  **Note:** This Makefile does *not* support having both the header and the
   source files mixed up (at least for libraries). You need to put your headers
   into a separate directory. This directory *may* be inside the `SRC_MAIN`
   directory.  
  It's always good practice to have an `include` directory next to your
   `SRC_MAIN` directory, have you headers in a subdirectory of this `include`
   directory and then add `-Iinclude` to your compiler flags.
* `LINK_DIRS`  
  A list of directories to search for when linking libraries.  
  (this option will not be already written in this section like the others,
   because you will rarely ever use it)
* `LINKS`  
  A list of libraries to link with.  
  **Note:** We *cannot* link any libraries when building a library ourself. The
   Makefile will throw an error if this variable is defined and `SOFTWARE` is
   set to `lib`
* `TEST`  
  The program to use to test all built test targets. Every test will be passed
   to this command, each with a `./` prefix added onto them.  
  If you don't have a program that can call tests like this, I recommend
   [utest-script](https://github.com/mfederczuk/utest-script). (also written by
   me)
* `MAIN`  
  The source file which contains the `main` function.  
  When linking object files to the test executables, this file will be excluded.  
  This variables is only needed when building an executable.
* `CCFLAGS` & `CXXFLAGS`  
  It's standard Makefile conventions  to use these variables as flags for the
   **C** and **C++** compiler, add include directories (`-I`), language
   standards (`-std=`) and warning flags like `-Wall` here.  
  Do *not* add compilation options like `-c`, output options like `-o` or
   linking options like `-l`
* `NO_COLOR`  
  By default, the messages the Makefile prints out will be colored. This can be
   disabled by setting the `NO_COLOR` variable to `1`.

Several variables will already have a value written in them. This is what *I*
 perceive to be best practice. Feel free to change them however you like.

## Usage ##

The usage is just like any other standard Makefile.  
The Makefile contains the rules `all`, `install`, `uninstall`, `clean` as well
 as the names of the targets and object files to build.  
For a full list of rules and what they do, see [Makefile Rules](#makefile-rules).

Compliant with **GNU** conventions, you can define the `DESTDIR` variable, to
 change the directory any files will be installed to.  
Setting it to `/tmp` will install any files to `/tmp/usr/local/...`.  
The subdirectories do not need to exist, the Makefile will create them for you.

Change where targets and headers are installed with the `prefix` and
 `exec_prefix` variables. By default, `prefix` will be `/usr/local` and
 `exec_prefix` will be `$(prefix)`.  
`bindir` is defined as `$(exec_prefix)/bin` and will be used to install
 executables.  
`includedir` is set to `$(prefix)/include` and is used to install the library
 headers.  
`libdir`'s default value is `$(exec_prefix)/lib` and is the location for the
 installed libraries.

Any of these variables can either be set/changed inside the Makefile or on the
 command line.

### Testing ###

*Tests are enabled by default, but are optional, if you don't want to use tests
 use the `SRC` variable instead of `SRC_MAIN`. Make sure to also remove the
 `SRC_TEST` variable.*

Every file in the directory saved in the `SRC_TEST` variable will be built to
 a test executable.  
The name of the executable will be the base name of the file (directory and
 extensions removed), plus pre- & suffixes.  
By default the prefix will be `$(exe_prefix)` and the suffix will be
 `_test$(exe_suffix)`. The value is saved inside the variables `test_prefix` and
 `test_suffix`.

The `test` rule will try to invoke every test by passing every executable, with
 a `./` prefix, to the `TEST` variable.  
As previously mentioned, [utest-script](https://github.com/mfederczuk/utest-script)
 is a good choice as a testing software as it is literally created for something
 like this.

If you're building a library, every test file will be linked with the static
 object files, so that you can actually test the content in the library.  
When building an executable the test executables will also be linked with the
 object files, the only file that will not be linked with is the file which
 contains the `main` function, which is stored inside the `MAIN` variable.

### Makefile Rules ###

The Makefile has a bunch of rules that you can target.  
You can build the object files and the targets, install and uninstall the
 targets and clean the object files and the targets.

Most of the time `make` and `make clean` will be enough for testing your
 software. When installing the software, use `make` and `sudo make install`.

* `all` (executable & library)  
  Default rule; builds the executable or the libraries
* `universe` (executable & library)  
  Builds the executable or the libraries and the tests (if tests are disabled,
   this is identical to `all`)

**building object files:**

* `objects` (executable & library)  
  When building an executable: Builds the static object files and stores them
   inside the `BIN` directory.  
  When building a library: Builds the shared and static object files and stores
   them inside the `BIN` directory
* `objects/shared` (library)  
  Builds the shared object files and stores them inside the `BIN` directory
* `objects/static` (library)  
  Builds the static object files and stores them inside the `BIN` directory
* `$(BIN)/`*\<object file\>* (executable & library)  
  Builds the specified object file individually and stores it inside the `BIN`
   directory.

**building targets:**

* `targets` (library)  
  Builds the shared & static object files and both the shared & static libraries.  
  The binaries are saved next to the Makefile
* `target` (executable)  
  Builds the static object files and the executable.  
  The binary is saved next to the Makefile
* *\<executable target\>*  (executable)  
  Builds the static object files and the executable.  
  The binary is saved next to the Makefile
* *\<shared library target\>* (library)  
  Builds the shared object files and the shared library individually.  
  The binary is saved next to the Makefile
* *\<static library target\>* (library)  
  Builds the static object files and the static library individually.  
  The binary is saved next to the Makefile

**building & invoking tests:**

* `tests` (executable & library)  
  Builds the tests.  
  The binaries are saved next to the Makefile
* *\<test target\>* (executable & library)  
  Builds the specified test individually and stores the binary next to the
   Makefile
* `test` (executable & library)  
  Invokes all tests by passing the binaries to the `$(TEST)` command

**installing targets & headers:**

* `install` (executable & library)  
  When building an executable: Builds the static object files and the executable
  and installs the binary into `$(DESTDIR)$(bindir)`.  
  When building a library: Builds the shared & static object files and the
   shared & static libraries and installs the binaries into `$(DESTDIR)$(libdir)`
   and the header directory into `$(DESTDIR)$(includedir)`
* `install/targets` (library)  
  Builds the shared & static object files and the shared & static libraries and
   installs the binaries into `$(DESTDIR)$(libdir)`
* `install/target` (executable)  
  Builds the static object files and the executable and installs the binary into
   `$(DESTDIR)$(bindir)`.
* `install/`*\<executable target\>* (executable)  
  Builds the static object files and the executable and installs the binary into
   `$(DESTDIR)$(bindir)`.
* `install/`*\<shared library target\>* (library)  
  Builds the shared object files and the shared library and installs the binary
   into `$(DESTDIR)$(libdir)`
* `install/`*\<static library target\>* (library)  
  Builds the static object files and the static library and installs the binary
   into `$(DESTDIR)$(libdir)`
* `install/headers` (library)  
  Installs the header directory into `$(DESTDIR)$(includedir)`

**uninstalling targets & headers:**

* `uninstall` (executable & library)  
  When building an executable: Removes the executable binary from
   `$(DESTDIR)$(bindir)`.  
  When building a library: Removes the shared and static library binaries from
   `$(DESTDIR)$(includedir)` and the header directory from `$(DESTDIR)$(includedir)`
* `uninstall/targets` (library)  
  Removes the shared and static library binaries from `$(DESTDIR)$(libdir)`
* `uninstall/target` (executable)  
  Removes the executable binary from `$(DESTDIR)$(bindir)`.
* `uninstall/`*\<executable target\>* (executable)  
  Removes the executable binary from `$(DESTDIR)$(bindir)`.
* `uninstall/`*\<shared library target\>* (library)  
  Removes the shared library binary from `$(DESTDIR)$(libdir)`
* `uninstall/`*\<static library target\>* (library)  
  Removes the static library binary from `$(DESTDIR)$(libdir)`
* `uninstall/headers` (library)  
  Removes the header directory from `$(DESTDIR)$(includedir)`

**cleaning object files, targets & tests:**

* `clean` (executable & library)  
  Removes all object files, the executable or the libraries and all tests
* `clean/objects` (executable & library)  
  Removes the object files in the `BIN` directory and `BIN` or any
   subdirectories if they became empty
* `clean/objects/shared` (library)  
  Removes the shared object files in the `BIN` directory and `BIN` or any
   subdirectories if they became empty
* `clean/objects/static` (library)  
  Removes the static object files in the `BIN` directory and `BIN` or any
   subdirectories if they became empty
* `clean/$(BIN)` (executable & library)  
  Removes the `BIN` directory
* `clean/$(BIN)/`*\<object file\>* (executable & library)  
  Removes the specific object file in the `BIN` directory and `BIN` or any
   subdirectories if they became empty
* `clean/targets` (library)  
  Removes both the shared and static library binary
* `clean/target` (executable)  
  Removes the executable binary
* `clean/`*\<executable target\>* (executable)  
  Removes the executable binary
* `clean/`*\<shared library target\>* (library)  
  Removes the shared library binary
* `clean/`*\<static library target\>* (library)  
  Removes the static library binary
* `clean/tests` (executable & library)  
  Removes all test targets
* `clean/`*\<test target\>* (executable & library)  
  Removes the specific test target

**other:**

* `_version`  
  Displays the current version of the Makefile template

## Contributing ##

Read through the [C/C++ Makefile Template Contribution Guidelines](./CONTRIBUTING.md)
 if you want to contribute to this project.

## License ##

[GNU GPLv3+](./LICENSE)

## Appendix A: File Extensions ##

These file extensions were taken from the **GCC** man page.

**C:**

* `c`
* `ii`

**C++:**

* `C`
* `cc`
* `cp`
* `ii`
* `c++`
* `cpp`
* `CPP`
* `cxx`
