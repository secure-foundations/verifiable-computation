# Pinocchio v0.5.3

The code included here implements the Pinocchio toolchain, as described in:

[Pinocchio: Nearly Practical Verifiable Computation](https://www.andrew.cmu.edu/user/bparno/papers/pinocchio.pdf)
Bryan Parno, Craig Gentry, Jon Howell, and Mariana Raykova
In Proceedings of the IEEE Symposium on Security and Privacy, May 21, 2013

It also includes support for the older QSP and QAP protocols from:

[Quadratic Span Programs and Succinct NIZKs without PCPs](https://www.andrew.cmu.edu/user/bparno/papers/quadratic.pdf)
Rosario Gennaro, Craig Gentry, Bryan Parno, and Mariana Raykova
In Proceedings of the IACR Eurocrypt Conference, May 2013
Originally published as Cryptology ePrint Archive, [Report 2012/215](http://eprint.iacr.org/2012/215), April, 2012

This code comes with all of the usual academic-prototype disclaimers:
it was written in a hurry, before a deadline, etc., and hence makes no 
claims to correctness, usability, or intelligibility.  Use at your own risk!

Related to the above, the interfaces for performance testing have received much
more extensive use and testing that the interfaces for external users, so
apologies in advance if the user interfaces are flaky.  Similarly, the Pinocchio 
protocols have received much more testing than the GGPR protocols, so not all
options will work with GGPR.

Finally, Pinocchio makes use of a high-speed cryptographic library internal to
Microsoft, which we cannot release.  Thus, this code release exposes a series
of hooks (see Field.h and Encoding.h) where you can plug in your own
elliptic-curve and field manipulation libraries (e.g., the [Pairing-Based
Cryptography library](http://crypto.stanford.edu/pbc/).  The current release
also includes a binary version of Pinocchio, so that you can use the same
executables we did.

Caveat: The binaries require the `--mem argument`, even though the help message
does not list it as mandatory.  

More information can be found on the Wiki.


# CODE WALKTHROUGH

- ccompiler: 
  Converts a program written in a subset of C into an arithmetic or
  Boolean circuit.  The function you wish to execute verifiably should be:
  void outsource(struct Input *input, struct Output *output)

  - external-code: 
    Run make here to download some additional libraries the compiler needs

  - input:
    Contains a number of sample programs we used to test Pinocchio.  To build
    them, run ../src/build-test-matrix.py.  Running "make -f make.matrix" should
    produce a build directory containing circuits and executables.  The specific
    apps and parameter settings built by build-test-matrix are governed by the
    settings in common/App.py.  

  - src
    Contains the main sources for the compiler plus some additional helper
    scripts (e.g., drawcirc.py can help you visualize the circuits you're
    producing, though it tends to drag if you go above 1000 gates).  The
    makefile produced by running build-test-matrix.py (as described above)
    should illustrate how to invoke the compiler directly.
 
- qap:
  Contains C++ code that takes a circuit as input, converts it to a QSP or QAP
  and then runs the various Pinocchio algorithms.  The circuit to QSP/QAP
  conversion should be a part of the compiler, but it lives here for historical
  reasons. 
  
  Arguments can be passed in via the command line or via a configuration file,
  thanks to the Boost program_options library, which you will need to acquire.
  We tested with version 1.51, but other versions are likely to work as well.
  Running the code with a -h flag will provide a summary of all of the various
  options you can use (many of which are only for debugging/performance
  testing).  

  The code was develop in Visual Studio, but it should be largely platform
  independent.  It uses a hash_map in one place; if your version of the STL
  doesn't include a hash_map, it can be replaced with a map at a ~log(N) perf 
  penalty.

  The main configuration file is Types.h, where you can enable various debugging
  flags and toggle between the GGPR protocols (using #define USE_OLD_PROTO) and
  the newer Pinocchio protocols.  

  The show starts in main.cpp, which handles configuration parsing and 
  dispatches everything to QSP or QAP routines.  The test.cpp and qsp-tests.cpp 
  files contain most of the high-level logic, including numerous functionality
  tests, many of which have atrophied, so use them at your own risk.

  The qap/windows directory contains a basic solution and project file, though
  they will not compile without cryptographic library support.

# VERSION HISTORY
- v0.5.3: Bug fix in NIZK input handling.  Reported by wacban via vc.codeplex.com.
- v0.5.2: Bug fix for the network client.  
- v0.5.1: Bug fix for the constant 1 input in NIZK instances.  Reported by Ahmed Kosba.
- v0.5:   Bug fix for the --verify option.  Reported by azerty123 via vc.codeplex.com.
- v0.4:   Minor compiler bug fixes.  Included the executable.
- v0.3:   Added proper support for equality testing.
- v0.2:   Included some additional files and fixed a few verification-related bugs.
- v0.1:   Initial release, source-code only.
