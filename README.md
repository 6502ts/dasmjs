# The dasm macro assembler (for JavaScript)

[![npm](https://img.shields.io/npm/v/dasm.svg)](https://www.npmjs.com/package/dasm)
[![Dependency Status](https://david-dm.org/zeh/dasmjs.svg)](https://david-dm.org/zeh/dasmjs)

This is an [emscripten](https://github.com/kripken/emscripten)-compiled version of the [dasm macro assembler](http://dasm-dillon.sourceforge.net/).

The dasm macro assembler transforms assembly code into 6502-compatible executable binary code. Since this is a JavaScript port of dasm, it allows that compilation process from JavaScript programs; more especifically, it can be used to create Atari VCS 2600 and Fairchild Channel F ROMs from a string containing dasm-compatible assembly source code.

In other words, it turns something like this:

```assembly
; Pick the correct processor type
        processor 6502
; Basic includes
        include "vcs.h"
        include "macro.h"
; Start address
        org $f000
; Actual instructions
start   SEI
        CLD
        LDX  #$FF
        TXS
        LDA  #$00
        ...
```

...into its equivalent byte code:

```assembly
f000 78
f001 d8
f002 a2 ff
f004 9a
f005 a9 00
...
```

Among other features, dasm sports:

* fast assembly
* several binary output formats available
* expressions using [] for parenthesis
* complex pseudo ops, repeat loops, macros, etc

This port of dasm was created so I could have dasm compiling working in a [vscode-dasm](https://github.com/zeh/vscode-dasm), my Visual Studio Code extension that aims to allow Atari development and debugging from within Visual Studio Code.

## Technical information

This package uses version 2.20.11 of dasm. It supports the following processor architectures:

* 6502 (and 6507)
* 68705
* 6803
* HD6303 (extension of 6803)
* 68HC11

This specific port was built on Linux (err, Windows 10 bash) from the dasm source using emscripten 1.37.0. Check the [dasm folder](https://github.com/zeh/dasmjs/tree/master/dasm) for the script that was used to compile `dasm.js`, including its pre/post-JS includes to wrap the code in a module function and return its results in a more usable way.

## Usage

Install:

```shell
npm install dasm --save
```

Import as a module:

```JavaScript
import dasm from "dasm"; // ES6
var dasm = require("dasm").default; // ES5
```

Finally, convert code to a binary data ROM. Instead of forcing developers to use a command line-like interface, the function that wraps the emscripten module provides a modern interface to dasm:

```JavaScript
// Read utf-8 assembly source
const src = "...";

// Run with the source
const result = dasm(src);

// Read the output as a binary (Uint8Array array)
const ROM = result.data;
```

### Advanced usage

Advanced options can be passed to the `dasm` call via an options parameter. For example:

```JavaScript
// Create a rom using the typical Atari VCS 4096-byte format
dasm(src, { format: 3 });

// Just create a rom without exporting symbols or lists
dasm(src, { quick: true });

// Typical assembly to create an Atari VCS ROM with built-in .h includes
dasm(src, { format: 3, quick: true, machine: "atari2600" });

// Pass original command-line parameters
dasm(src, { parameters: "-f3 -p2 -v4 -DVER=5" });
```

These are all the options currently parsed:

* `format`: binary output format. Dictates the size and arrangement of the generated ROM.
  * `1` (default): output includes a 2-byte origin header.
  * `2`: random access segment format. Output is made of chuncks that include a 4-byte origin and length header.
  * `3`: raw format. Just the data, no headers.
* `quick`: boolean. If set to `true`, don't export any symbol and pass list as part of its returned data. Defaults to false.
* `parameters`: string. List of switches passed to dasm as if it was being called from the command line.
* `include`: key-value object. This is a list of files that should be made available for the source code to `include`. The key contains the filename, and the value, its content.
* `machine`: target machine. Similarly to dasm's `-I` switch, this picks a list of (embedded) files to make available to the `include` command.
  * `"atari2600"`: includes dasm's own `atari2600/macro.h` and `atari2600/vcs.h` files.
  * `"channel-f"`: includes dasm's own `channel-f/macro.h` and `channel-f/ves.h` files.

Check [the dasm documentation](https://github.com/zeh/dasmjs/blob/master/dasm/src/doc/dasm.txt) for a list of all command-line switches available, for more information on binary formats, and for a list of all macros available.

### Returned object

The object returned by the `dasm` function has more than just a binary ROM. This is what's available:

* `data`: `Uint8Array`. The exported ROM, as a list of integers.
* `output`: `string[]`. All data written by dasm to `stdout`.
* `list`: `IList[]`. A list of all lines available in the source code, and their parsed info (address, bytecode, comments, command, etc).
* `listRaw`: `string`. The raw output of the list file (equivalent to the `-L` switch).
* `symbols`: `ISymbol[]`. A parsed list of all symbols (labels and constants) defined by the source code.
* `symbolsRaw`: `string`. The raw output of the symbols file (equivalent to the `-s` switch).

### More information

TypeScript definitions are included with this distribution, so TypeScript projects can use the module and get type checking and completion for all `dasm` calls. Non-TypeScript JavaScript developers using Visual Studio Code will also benefit from auto-completion without any change thanks to VSC's [Automatic Type Acquisition](http://code.visualstudio.com/updates/v1_7#_better-javascript-intellisense).

## Todo

* More examples, including on how to include files
* Get exitcode from executable in case of errors?
* Fix: hanging when files are not found?
* More tests: all options
* Run as a worker?
* Command-line package? (`dasm-cli`)

Contributions are welcome.

## Changelog

Check [the release list](https://github.com/zeh/dasmjs/releases) for a list of what has changed in every new version.

## Acknowledgements

The dasm macro assembler was created by by Matthew Dillon. It was further augmented by Olaf "Rhialto" Seibert, Andrew Davie, and Peter H. Froehlich.

## License

This follows dasm itself and uses the [GNU Public License v2.0](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html).