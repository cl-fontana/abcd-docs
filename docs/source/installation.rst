.. _installation:

============
Installation
============

In principle ABCD could run on any `POSIX-compliant <https://en.wikipedia.org/wiki/POSIX>`_ system, but it is mainly used on Linux distributions.
The modules that do not interface with some hardware are the most portable, on the other hand the hardware interfaces depend on vendors' drivers that might not be available on many systems.

.. _download:

Download
--------

The `official repository <https://github.com/ec-jrc/abcd/>`_ offers the download of pre-compiled packages under the `releases page <https://github.com/ec-jrc/abcd/releases>`_.
"Assets" section of each release offers the links to the packages.
Three packages are offered and they are meant for Ubuntu 24.04 LTS:

* `abcd-core_<version number>_amd64.deb` That installs the core system of ABCD, for this tutorial it is sufficient to install this package;
* `abcd-abcd_<version number>_amd64.deb` That installs the support for CAEN digitizers, install it only if you desire to use such digitizers;
* `abcd-absp_<version number>_amd64.deb` That installs the support for SP Devices digitizers, install it only if you desire to use such digitizers.

.. _packages-installation:

Packages installation
---------------------

Use the standard Ubuntu tools to install packages::

    sudo dpkg -i abcd-core_<version number>_amd64.deb

If there were some missing packages which ABCD depends on, install them with::

    sudo apt install --fix-broken

The supported distribution for the packages is: Ubuntu 24.04 LTS.
The installation places the ABCD modules, tools and scripts in the ``$PATH``.
Examples scripts, configurations, libraries and data are stored in the directory ``/usr/share/abcd/``.
It is advisable to copy the examples to a local directory, thus not working directly in  ``/usr/share/abcd/``.


Manual installation
-------------------

A manual installation is needed only if your system is not supported, or you want to modify the source code of ABCD, and it is not necessary for following the tutorial.
The list of required libraries and tools is:

* `git <https://git-scm.com/>`_, for fetching and updating the source code;
* `cmake <https://cmake.org/>`_, for building the framework;
* `tmux <https://github.com/tmux/tmux/wiki>`_, for running ABCD in the background;
* `clang <https://clang.llvm.org/>`_ or `gcc <https://clang.llvm.org/>`_, for compiling the framework;
* the `ZeroMQ messaging library <https://zeromq.org/>`_, for data delivery between modules;
* the `GNU Scientific Library <https://www.gnu.org/software/gsl/>`_, for some useful functions;
* `JsonCpp <https://github.com/open-source-parsers/jsoncpp>`_ **and** `Jansson <https://github.com/akheron/jansson>`_, for decoding and encoding JSON messages;
* `Python 3 <https://www.python.org/>`_, for analysis and automation scripts, together with the libraries: `PyZMQ <https://github.com/zeromq/pyzmq>`_, `NumPy <https://numpy.org>`_, `SciPy <https://scipy.org/>`_ and `Matplotlib <https://matplotlib.org/>`_;
* `Node.js <https://nodejs.org/>`_ 12 or later, with NPM, for running the web interface;
* `zlib <https://zlib.net/>`_ and `libbzip2 <https://www.sourceware.org/bzip2/>`_, for data compression;

If the hardware interfacing modules need to be compiled, then their libraries are needed before their compilation step.
Check the section :numref:`vendors-libraries` for some tips.

.. _manual-installation-compilation:

Compilation
```````````

Use ``git`` to download the repository::

    user-tutorial@abcd-tutorial:~$ git clone https://github.com/ec-jrc/abcd.git
    Cloning into 'abcd'...
    remote: Enumerating objects: 2084, done.
    remote: Counting objects: 100% (418/418), done.
    remote: Compressing objects: 100% (182/182), done.
    remote: Total 2084 (delta 256), reused 376 (delta 236), pack-reused 1666
    Receiving objects: 100% (2084/2084), 67.80 MiB | 7.38 MiB/s, done.
    Resolving deltas: 100% (1291/1291), done.

Then, use CMake to build ABCD from its directory::

    cd abcd
    cmake -S . -B build
    cmake --build build --parallel 8

The installation can be launched with CMake::

    sudo cmake --install build

If installing manually on linux with ``cmake --install ...``, then ``ldconfig`` should be launched after the installation.
``ldconfig`` configures the cache of the system-wide libraries, so that the waan libraries can be located by ``waan``.

Modules interfacing with hardware are not built by default, as they depend on specific vendors' libraries.
To compile the specific modules use the ``-D BUILD_<MODULE>=ON`` option in the configuration phase, as in::

    cmake -S . -B build -D BUILD_ABSP=ON -D BUILD_ABCD=ON
    cmake --build build --parallel 8
    sudo cmake --install build

Ubuntu package(s) generation and installation
`````````````````````````````````````````````

The Ubuntu packages may be generated with CMake and CPack.
The default prefix for CMake on UNIX platforms is ``/usr/local``, but it should be changed to ``/usr`` at the configuration phase to generate a proper ``deb`` package.
Some scripts need to be configured with the proper prefix and this is done at the configuration phase.
There is no need to run ``ldconfig`` after the installation, as it is automatically run by the ``postinst`` script of the ``deb`` package.

To generate and install the ``deb`` package::

    cmake -S . -B build --install-prefix /usr
    cmake --build build --parallel 8
    cd build/
    cpack
    sudo dpkg -i abcd-core-...

Hardware interfacing modules need to be explicitly enabled as in the standard build.
If enabled, separate packages are generated for each hardware module, as in::

    cmake -S . -B build -D BUILD_ABSP=ON -D BUILD_ABCD=ON --install-prefix /usr
    cmake --build build --parallel 8
    cd build/
    cpack
    sudo dpkg -i abcd-core-...
    sudo dpkg -i abcd-absp-...
    sudo dpkg -i abcd-abcd-...

If there were some missing packages on which ABCD depends on, install them with::

    sudo apt install --fix-broken

.. warning::
    
    In the installation phase of the dependencies of node.js, sometimes ``npm`` complaints about security issues and suggests to launch ``npm audit``.
    This is a normal warning that can be ignored, keeping in mind that ABCD does not guarantee **any** kind of security.
    ABCD instances should be run in a protected environment in which malicious users can do no harm.

.. _vendors-libraries:

Vendors libraries
-----------------

For the installation details of the hardware, you will have to refer to the vendors' websites.
Some suggestions are provided here.

CAEN digitizers
```````````````

CAEN digitizers are interfaced by the ``abcd`` module.
For their support, the following CAEN libraries for linux are required:

* `CAEN VMELib <https://www.caen.it/products/caenvmelib-library/>`_;
* `CAEN Comm <https://www.caen.it/products/caencomm-library/>`_;
* `CAEN Digitizer <https://www.caen.it/products/caendigitizer-library/>`_;
* `CAEN DPP <https://www.caen.it/products/caendpp-library/>`_;
* `CAEN USB driver <https://www.caen.it/download/?filter=V1718>`_ (in the software downloads of the V1718).

It seems that the CAEN USB kernel module needs to be recompiled every time that the kernel is updated.
Therefore if the system updates the kernel, we suggest to reinstall the CAEN libraries.

SP Devices
``````````

SP Devices digitizers are interfaced by the ``absp`` module.
The `ADQAPI <https://www.spdevices.com/documents/user-guides/68-adqapi-reference-guide/file>`_ for linux is required for the SP Devices digitizers, refer to the `official vendor's page <https://www.spdevices.com/products/software>`_.

Digilent
````````

The `Digilent Analog Discovery 2 (AD2) <https://digilent.com/shop/analog-discovery-2-100ms-s-usb-oscilloscope-logic-analyzer-and-variable-power-supply/>`_ is interfaced by the ``abad2`` module.
For the support, the requirements for linux are:

* `WaveForms <https://store.digilentinc.com/digilent-waveforms/>`_ software for its SDK;
* `Adept 2 <https://reference.digilentinc.com/reference/software/adept/>`_ for the hardware communication.

Red Pitaya
``````````

The Red Pitaya's STEMlab boards have the libraries already installed, refer to the official `documentation <https://redpitaya.readthedocs.io/en/latest/>`_.
ABCD should be downloaded in the STEMlab itself and run from its local filesystem.
The module providing support to the hardware is the ``abrp`` module.