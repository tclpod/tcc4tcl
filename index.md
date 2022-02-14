tcc4tcl - Progamming with C in Tcl
==================================

[tcc]: http://bellard.org/tcc/

tcc4tcl (Tiny C Compiler for Tcl) is a Tcl extension that provides an interface to [TCC].


## Quick Example

```tcl
package require tcc4tcl

::tcc4tcl::cproc test {int i} int {
    return (i + 42); 
}

puts [test 1]
```

* `tcc4tcl::cproc` is used to define Tcl command with C
* `{int i}` is arguments
* `int` after `{int i}` is return value type

[More examples ...](example.md)

## High-Level API Overview

```tcl
package require tcc4tcl

tcc4tcl::cproc <procName> <argList> <returnType> ?<code>?

set handler [tcc4tcl::new]
$handler cproc <procName> <argList> <returnType> ?<code>?
$handler go
```

tcc4tcl::new
------------

Creates a new TCC interpreter instance.

Synposis:

        tcc4tcl::new ?<outputFile> ?<packageNameAndVersionAsAList>??

Returns an opaque handle which is also a Tcl command to operate on.

If neither `<outputFile>` nor `<packageNameAndVersionAsAList>` are specified, compilation (which happens when [$handle go] is called) is performed to memory.

If only `<outputFile>` is specified then an executable is written to the file named.

If `<packageNameAndVersionAsAList>` is also specified then a Tcl extension is written as a shared library (shared object, dynamic library, dynamic linking library) to the file named.  The format is a 2 element list, the first is the name of the package and the second is the version number.

Examples:

  1.  Create a handle that will compile to memory:
    1.  `set handle [tcc4tcl::new]`
  2.  Create a handle that will compile to an executable named "myProgram":
    2.  `set handle [tcc4tcl::new myProgram]`
  3.  Create a handle that will compile to a shared library named "myPackage" with the package name "myPackage" and version "1.0":
    3.  `set handle [tcc4tcl::new myPackage "myPackage 1.0"]`

$handle cproc
-------------
Creates a Tcl procedure that calls C code.

Synoposis:

        $handle cproc <procName> <argList> <returnType> ?<code>?

  1. `<procName>` is the name of the Tcl procedure to create
  2. `<argList>` is a list of arguments and their types for the C function;
    2. The list is in the format of: type1 name1 type2 name2 ... typeN nameN
    2. The supported types are:
       2. Tcl_Interp*: Must be first argument, will be the interpreter and the user will not need to pass this parameter
       2. int
       2. long
       2. float
       2. double
       2. char*
       2. Tcl_Obj*: Passes the Tcl object in unchanged
       2. Tcl_WideInt
       2. void*
  3. `<returnType>` is the return type for the C function
    3. The supported types are:
       3. void: No return value
       3. ok: Return TCL\_OK or TCL_ERROR
       3. int
       3. long
       3. float
       3. double
       3. Tcl_WideInt
       3. char*: TCL\_STATIC string (immutable from C -- use this for constants)
       3. string, dstring: return a (char*) that is a TCL\_DYNAMIC string (allocated from Tcl\_Alloc, will be managed by Tcl)
       3. vstring: return a (char*) that is a TCL\_VOLATILE string (mutable from C, will be copied be Tcl -- use this for local variables)
       3. default: Tcl\_Obj*, a Tcl Object
  4. `<code>` is the C code that comprises the function.  If the `<code>` argument is omitted it is assumed there is already an implementation (with the name specified as `<procName>`, minus any namespace declarations) and this just creates the wrapper and Tcl command.

$handle cwrap
-------------
Creates a Tcl procedure that wraps an existing C function

Synoposis:

        $handle cwrap <procName> <argList> <returnType>

Notes:

This only differs from the `cproc` subcommand with no `<code>` argument in that it creates a prototype before referencing the function.

$handle ccode
-------------
Compile arbitrary C code.

Synopsis:

        $handle ccode <code>

$handle tk
----------
Request that Tk be used for this handle.

Synposis:

        $handle tk

$handle linktclcommand
----------------------
Create a Tcl command that calls an existing C command as a Tcl command.

Synopsis:

        $handle linktclcommand <CSymbol> <TclCommandName> ?<ClientData>?

$handle add\_include\_path
------------------------
Search additional paths for header files

Synopsis:

        $handle add_include_path <dir...>

$handle add\_library\_path
------------------------
Search additional paths for libraries

Synopsis:

        $handle add_library_path <dir...>

$handle add\_library
-------------------
Link to an additional library

Synopsis:

        $handle add_library <library...>

$handle code
------------
Return text of what code will be compiled when the _go_ subcommand is called.

Synposis:

        $handle code

$handle go
----------
Execute all requested operations and output to memory, an executable, or DLL.

Once this command completes the handle is released.

Synopsis:

        $handle go

See also [Low-Level API](wiki/Low-Level API)
