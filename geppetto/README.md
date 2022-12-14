
# Geppetto v0.1

The code included here implements the Geppetto toolchain, as described in:

[Geppetto: Versatile Verifiable Computation](https://www.andrew.cmu.edu/user/bparno/papers/geppetto.pdf)
Craig Costello, C?dric Fournet, Jon Howell, Markulf Kohlweiss, 
Benjamin Kreuter, Michael Naehrig, Bryan Parno, Samee Zahur 
In Proceedings of the IEEE Symposium on Security and Privacy, May 18, 2015

The underlying cryptographic protocol originates in:

[Pinocchio: Nearly Practical Verifiable Computation](https://www.andrew.cmu.edu/user/bparno/papers/pinocchio.pdf)
Bryan Parno, Craig Gentry, Jon Howell, and Mariana Raykova
In Proceedings of the IEEE Symposium on Security and Privacy, May 21, 2013

And both use the Quadratic Arithmetic Program encoding from:

[Quadratic Span Programs and Succinct NIZKs without PCPs](https://www.andrew.cmu.edu/user/bparno/papers/quadratic.pdf)
Rosario Gennaro, Craig Gentry, Bryan Parno, and Mariana Raykova
In Proceedings of the IACR Eurocrypt Conference, May 2013
Originally published as Cryptology ePrint Archive, [Report 2012/215](http://eprint.iacr.org/2012/215), April, 2012

This code comes with all of the usual academic-prototype disclaimers:
it was written in a hurry, before a deadline, etc., and hence makes no 
claims to correctness, usability, or intelligibility.  Use at your own risk!

Finally, Geppetto's underlying crytographic machinery (FFLib) makes use of a
high-speed cryptographic library internal to Microsoft, which we cannot release.
Thus, this code release exposes a series of hooks (see Field.h and Encoding.h)
where you can plug in your own elliptic-curve and field manipulation libraries
(e.g., the [Pairing-Based Cryptography library](http://crypto.stanford.edu/pbc/).
The current release also includes FFLib as a DLL, so you can make changes to the 
F# code and compile a working executable.

# CODE WALKTHROUGH

Everything lives within the compiler/ directory.

- input: 
  Contains a number of sample programs we used to test Geppetto.  
  To run these examples, you'll need to download and install a copy of clang,
  e.g., from:
    http://llvm.org/releases/download.html
  The cygwin version of clang is _not_ compatible.  We've tested with
  LLVM versions 3.5.0, 3.5.2, and 3.6.2.

  The examples are all driven via the input Makefile, and generate all
  their intermediate files in the input/build directory. For example,
  running "make all" will run a number of basic regression tests.

  See the Makefile for a description and illustration of the
  Geppetto compiler options. 

- src
  Contains the main sources for the compiler 

  - arith
    Source code for the Cocks-Pinch curves we employ.  Also contains a BN curve
    implementation, which we do not currently use, except for testing.

  - arith-qap
    A version of the arith library above adapted to be QAP friendly.
    Compiles both with clang and with Geppetto.  Used for verifying cryptographic
    code, e.g., signatures or bootstrapped verification.

  - cpp
    Code for FFLib, our core cryptographic routines, including code for key,
    commitment, and proof generation and verification.

  - fsharp
    Compiler front end and logic for converting C code into QAPs.

- windows
  Contains a basic solution and project file for compiling the F# code against
  FFLib.  To build FFLib itself requires you to supply a cryptographic library
  that implements field, polynomial FFT, and BN elliptic curve operations.

- bin
  Contains FFLib as a DLL.  The F# code compiles against this DLL and the
  resulting executable will appear in this directory.

# COMPILING

Before compiling the F# code, you'll need the [PowerPack for FSharp
3.0](https://www.nuget.org/packages/FSPowerPack.Community), .NET 4.x, and
Visual Studio 2013.  

Once you have the PowerPack, in `./compiler/input` run `make llvm`.
That will create the necessary `LLVM_{grammar,syntax}.fs` files using
fsyacc and fslex. If you get a strange exception about FSharp.Core,
look below under Redirecting FSharp.Core

From this point, you should be able to compile Geppetto from the
Visual Studio solution (in Release mode by default, or in Debug mode
for debugging F# source code).

As mentioned above, to use Geppetto, you'll need to download and
install a copy of clang.

# Redirecting FSharp.Core
At present, FSharpPowerPack appears to work only with version 4.3.0.0
of the FSharp.Core runtime library. So you might get an error such as:
```
  $ make llvm
  cd ../src/fsharp ; /cygdrive/c/Program\ Files\ \(x86\)/FSharpPowerPack-4.0.0.0/bin/fsyacc.exe LLVM_grammar.fsy --module LLVM_grammar

Unhandled Exception: System.IO.FileNotFoundException: Could not load file or assembly 'FSharp.Core, Version=4.3.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies. The system cannot find the file specified.
   at <StartupCode$fsyacc>.$FSharp.PowerPack.FsYacc.Driver.main@()
Makefile:49: recipe for target '../src/fsharp/LLVM_grammar.fs' failed
make: *** [../src/fsharp/LLVM_grammar.fs] Error 127
```

If you do, chances are, you do not have the right version of FSharp.Core. The
way to fix this is to create a "binding redirection", that tells F# to look for
the newer version.  You do this by following the instructions below, originally
from a helpful [blog post](http://blog.ploeh.dk/2014/01/30/how-to-use-fsharpcore-430-when-all-you-have-is-431/)

Essentially, create two identical config files named "fsyacc.exe.config" and 
"fslex.exe.config" in the installation folder for FSharpPowerPack (in my case,
it was `C:\Program Files (x86)\FSharpPowerPack-4.0.0.0\bin`).  Paste the following
xml code in these two files:
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="FSharp.Core"
                          publicKeyToken="b03f5f7f11d50a3a"
                          culture="neutral"/>
        <bindingRedirect oldVersion="4.3.0.0"
                         newVersion="4.3.1.0"/>
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
```
You might have to adjust `newVersion` according to what you are currently using. 
After this, the `make llvm` step mentioned above should work. 


# ADDITIONAL NOTES

- The Makefile includes a BLT flag that indicates which build to run.  By default,
  it will try to run the Release build, so be sure you've built the Release build
  in Visual Studio first.

- The energy-saving circuits described in the Geppetto paper are only partially
  integrated at this time.

- Geppetto currently requires that every variable be initialized---otherwise, the
  interpreter reports an error e.g. as one tries to add an integer to an undefined value.

- Geppetto does not provide support for pointer arithmetic on unknown variables.

- Geppetto demands that its I/O be actually used in the computation.

- In whole-program mode, the main function must call `init()` before using
  any bank or outsourced call, and similarly main must call `init_BOOT()` before
  using any boot-level bank or outsourced call.

# VERSION HISTORY
- v0.1: Initial release, source-code plus executable.
