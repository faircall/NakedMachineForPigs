Amnesia: A Machine For Pigs Source Code
=======================

Code
----

Amnesia: A Machine For Pigs has been upgraded for GNU/Linux and macOS systems.

* Old-style C++ class declarations have been updated.
* All dependencies are now using their modern equivalents (except AngelScript) which is on 2.24.1 - this is due to the fact that this version is perfectly 64bit compatible and has compilable sources.  The API for the latest versions is not compatible and changing the script engine on such a complex title is bound to introduce regressions.
* Newton has NOT been upgraded and is still the 2.08 version from The Chinese Room.  I attempted for a few hours to take versions and fixes from various Amnesia TDD forks but none worked with A Machine for Pigs and each one crashed the game immediately.  2.08 is no longer available on the internet and I'll have to do some serious debugging to get later versions working.
* CMake project is buildable from scratch with no further setup.
* All CMake projects have been upgraded to remove deprecation notices.

Please do not forget to copy the lib64 content into your game folder as these will be needed.

Building
--------

Go into the directory you cloned this to and perform a 'mkdir build' and 'cmake ..'

All dependencies are included here and so after it finished the setup, run 'make'.  Then grab a beverage because it takes about 20 minutes.

Please note, that this project deals only with GNU/Linux and macOS builds.  Windows users aren't serviced here, sorry.

License Information
-------------------
All code is under the GPL Version 3 license. Read the LICENSE file for terms of use of the license.
