.. _installation:

============
Installation
============

In principle ABCD could run on any `POSIX-compliant <https://en.wikipedia.org/wiki/POSIX>`_ system, but it is mainly used on Linux distributions.
The modules that do not interface with some hardware are the most portable, on the other hand the hardware interfaces depend on vendors' drivers that might not be available on many systems.

.. _download:

Download
--------

The `git <https://git-scm.com/>`_ is required to download the latest version of the ABCD source code.
The ABCD repository can be downloaded as::

    user-tutorial@abcd-tutorial:~$ git clone https://github.com/ec-jrc/abcd.git
    Cloning into 'abcd'...
    remote: Enumerating objects: 2084, done.
    remote: Counting objects: 100% (418/418), done.
    remote: Compressing objects: 100% (182/182), done.
    remote: Total 2084 (delta 256), reused 376 (delta 236), pack-reused 1666
    Receiving objects: 100% (2084/2084), 67.80 MiB | 7.38 MiB/s, done.
    Resolving deltas: 100% (1291/1291), done.

This command will create a new directory called ``abcd`` with the whole source code.
It is also possible to manually download a zip file with the source code from: <https://github.com/ec-jrc/abcd/archive/refs/heads/main.zip>
But the method with git is the preferred one, as it will make easier to keep the source code updated.

.. note::

    The rest of the installation instructions assume that ABCD was downloaded in ``~/abcd/``.

.. _automatic-installation:

Automatic installation
----------------------

There is an automatic `installation script <https://github.com/ec-jrc/abcd/blob/main/install_dependencies.sh>`_, that tries to identify the current Linux distribution, install the required packages and compile the framework.
The supported distributions for the installation script are: Ubuntu 20.04 LTS, Ubuntu 22.04 LTS, and Rocky Linux 8.

.. note::

    The installation script runs some commands with ``sudo`` and thus the user should have the permissions for that.

.. note::

    The following software is required in order to run the installation: `Linux Standard Base (LSB) utilities <https://en.wikipedia.org/wiki/Linux_Standard_Base>`_, used only in the installer script to detect the Linux distribution.
    It can be installed with:
    
    * On Debian and Ubuntu: ``apt-get install lsb-release``
    * On CentOS and Fedora: ``dnf install redhat-lsb``

Run the installation script in the folder in which ABCD was downloaded as::
   
    user-tutorial@abcd-tutorial:~$ cd abcd
    user-tutorial@abcd-tutorial:~/abcd$ ./install_dependencies.sh
    ######################
    # Kernel name: Linux #
    ######################
    #######################
    # Linux system found! #
    #######################
    ####################################################
    # Detected distribution: Ubuntu; major release: 22 #
    ####################################################
    #######################
    # Upgrading system... #
    #######################
    [sudo] password for user-tutorial: 
    Hit:1 http://fr.archive.ubuntu.com/ubuntu jammy InRelease                                                                                                                         
    Get:2 http://fr.archive.ubuntu.com/ubuntu jammy-updates InRelease [114 kB]                                                                                                        

        (...)

    ##################################################################
    # Installing required packages, for Ubuntu 22 Jammy Jellyfish... #
    ##################################################################
    Reading package lists... Done
    Building dependency tree... Done
    Reading state information... Done

        (...)

    #########################################
    # The required packages were installed. #
    #########################################
    #####################
    # Compiling ABCD... #
    #####################
    make -C dasa; make -C chafi; make -C cofi; make -C wafi; make -C wadi; make -C enfi; make -C pufi; make -C gzad; make -C unzad; make -C fifo; make -C waps; make -C waph; make -C waan; make -C spec; make -C tofcalc; make -C replay; make -C convert;
   
        (...)
   
    #############################################
    # Installing the dependencies of node.js... #
    #############################################

        (...)

    #############################
    # Installation successfull! #
    #############################

In this preview the wordy parts of the installation were abridged for the sake of simplicity.
If any of these steps fail, you should refer to section :ref:`manual-installation` for a more detailed help.

.. warning::
    
    In the installation phase of the dependencies of node.js, oftentimes npm complaints about security issues and suggests to launch ``npm audit``.
    This is a normal warning that can be ignored, keeping in mind that ABCD does not guarantee **any** kind of security.
    ABCD instances should be run in a protected environment in which malicious users can do no harm.


.. warning::
    
    The hardware interfacing modules are not compiled, as they depend on specific libraries that might not be installed.
    The user should compile the suitable modules for the hardware (*e.g.* ``abcd``, ``abad2``, ``abps5000a``, ``abrp``, or ``absp``).
    See section :ref:`vendors-libraries` for suggestions on installing the vendors' libraries.

.. _manual-installation:

Manual installation
-------------------

If the installation script (see :ref:`automatic-installation`) fails or the system is not supported, it is still possible to manually install the dependencies.
The list of required libraries and tools is:

* `git <https://git-scm.com/>`_, for fetching and updating the source code;
* `tmux <https://github.com/tmux/tmux/wiki>`_, for running ABCD in the background;
* `clang <https://clang.llvm.org/>`_ or `gcc <https://clang.llvm.org/>`_, for compiling the framework;
* the `ZeroMQ messaging library <https://zeromq.org/>`_, for data delivery between modules;
* the `GNU Scientific Library <https://www.gnu.org/software/gsl/>`_, for some useful functions;
* `JsonCpp <https://github.com/open-source-parsers/jsoncpp>`_ **and** `Jansson <https://github.com/akheron/jansson>`_, for decoding and encoding JSON messages;
* `Python 3 <https://www.python.org/>`_, for analysis and automation scripts, together with the libraries: `PyZMQ <https://github.com/zeromq/pyzmq>`_, `NumPy <https://numpy.org>`_, `SciPy <https://scipy.org/>`_ and `Matplotlib <https://matplotlib.org/>`_;
* `Node.js <https://nodejs.org/>`_ 12 or later, with NPM, for running the web interface;
* `zlib <https://zlib.net/>`_ and `libbzip2 <https://www.sourceware.org/bzip2/>`_, for data compression;

Post-dependecies installation
`````````````````````````````

When the dependecies are installed the ABCD system should be compiled.
The default compiler is clang. It is also possible to change the compiler to gcc modifying the file `common_definitions.mk <https://github.com/ec-jrc/abcd/blob/main/common_definitions.mk>`_ in the abcd main directory.
The global Makefile compiles the whole system just by running in the abcd main directory as::

    user-tutorial@abcd-tutorial:~/abcd$ make
    make -C dasa; make -C chafi; make -C cofi; make -C wafi; make -C wadi; make -C enfi; make -C pufi; make -C gzad; make -C unzad; make -C fifo; make -C waps; make -C waph; make -C waan; make -C spec; make -C tofcalc; make -C replay; make -C convert;
   
        (...)

.. warning::

    The hardware interfacing modules are not compiled, as they depend on specific libraries that might not be installed.
    The user should compile the suitable modules for the hardware (*e.g.* ``abcd``, ``abad2``, ``abps5000a``, ``abrp``, or ``absp``).
    See section :ref:`vendors-libraries` for suggestions on installing the vendors' libraries.

The web-based user interface requires some modules for node.js that must be installed independently.
From the abcd main directory::

    user-tutorial@abcd-tutorial:~/abcd$ cd wit/
    user-tutorial@abcd-tutorial:~/abcd/wit$ npm install

These will download all the required packages for the Node.js server and install them locally in the ``wit/`` folder.

.. warning::
    
    In the installation phase of the dependencies of node.js, oftentimes npm complaints about security issues and suggests to launch ``npm audit``.
    This is a normal warning that can be ignored, keeping in mind that ABCD does not guarantee **any** kind of security.
    ABCD instances should be run in a protected environment in which malicious users can do no harm.

Once all the packages have been installed the system is ready to be run from the abcd main folder.

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
For the SP Devices digitizers, the `ADQAPI <https://www.spdevices.com/documents/user-guides/68-adqapi-reference-guide/file>`_ for linux is required, refer to the `official vendor's page <https://www.spdevices.com/products/software>`_.

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