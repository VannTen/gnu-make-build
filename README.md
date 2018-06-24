# Make Build System

## Design Goals

The purpose of this project is to implement a build system based on GNU Make,
by using the functionnal properties of the GNU Make syntax to create templates
and abstract the rules redaction.

## Code structure

This project will expect each directory of a project to have one and only one
target : every source file in one directory are expected to be linked together,
in an executable or shared library, or contained together in a static library.

## Interface

### Base

This systems will expect each source directory to contain a Srcs.mk files,
which shall define the following GNU Make variables :
- `$(this)_SRC` : shall contain the list of source files
- `$(this)_TARGET` : shall contain the name of the executable or library for
  that directory
- `$(this)_DEPENDENCIES` : shall contain the list of library that the target
  needs (to link against or to add to its derived dependencies).

### Others

The reste of the interface is yet to be thought.


## Targeted features

This build system aims to support :
- GNU Make Parallel execution
- Multi architectures builds and cross compilation
- Multi options builds (debug, optimized, etc) / eventually, the system
  could compute a hash of the final options used
- All the GNU Make standard targets and variables for GNU projects
- Delete old artifacts when target or list of source file changes
- Auto generation of of dependencies for header or similar
- Simple namespacing for depencies on libs

Other features (to think about) :
- Deal with generated code


## Implementation


To implement these features, this project will use several principles.

### Files category

The build system shall distinguish several categories of files :
- source files : quite transparent. These files are 'read-only' for the
  build system, and are the project source. The Srcs.mk files in each
  directory are also considered source files (though they might eventually
  be bootstrapped if they are not present for a project (to think about))
- architecture specific files / binary files : these are build artifacts
  which are different for each architecture, like object files.
  This is mostly binary files.
- architecture independant files : these are build artifacts which are the
  same regardless of the architecture, like generated code files.
These three kinds of files will be separated.

### Output structure

#### Binary files

Binary files are different based on compilation options. Hence, the build system
shall produce different sub build tree for different compilation options. 

To achieve this, the build system will map each options category (such as :
architecture, debug level, optimization level) to a directory level ; and each
alternative for each of those category to a sub directory at that level.

Example :

Let's suppose we have the architecture options category, which is either `amd64`
or `x86`, and the debug level category, which is either `g0` or `g1`.
The directory structure shall look like this :
```
.
├── amd64
│   ├── g0
│   └── g1
└── x86
    ├── g0
    └── g1
```
And each alternative will its own compilation options, so files in `amd64/g0`
are build with `--arch amd64 -g0` and those in `x86/g1` with `--arch amd64 -g1`.
The order of that directory hierarchy shall be left to the user, but the
interface remain to be defined.
