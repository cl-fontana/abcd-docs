===================================================
General purpose waveforms analysis: ``waan`` module
===================================================

``waan`` is a general purpose waveforms analysis software.
It allows users to define their own waveforms analysis routines, without worrying about the inner workings of the ABCD data acquisition system.
The user is responsible to write two libraries in C99, that take a waveform as input and outputs one or more processed event.
The user functions can be modified and reloaded at runtime, simplifying the development phase.

Module structure
----------------

.. code-block:: none
    :name: diagram-simple-connections
    :caption: Diagram of the structure of the waveform analysis module ``waan``.

                                                       +-------------------+    
                                                       | +-------------------+  
                                                       | | +-------------------+
    +----------+                                       | | | user libraries    |
    | Waveform | <== In the form of an                 +-| | (may be different |
    +----------+     event_waveform object,              +-| for each channel) |
         |           see include/events.h                  +-------------------+
         v                                                           |          
    +----------------------+    +--------+    +------------------+   |          
    | user function:       | <- | user   | <- | user function:   | <-+          
    | timestamp_analysis() |    | config |    | timestamp_init() |   |          
    +----------------------+    +--------+    +------------------+   |          
         |             |                                             |          
         v             v                                             |          
    +----------+   +-------+                                         |          
    | Waveform |   | Event | <== In the form of an                   |          
    +----------+   +-------+     event_PSD object,                   |          
         |             |         see include/events.h                |          
         v             v                                             |          
    +----------------------+    +--------+    +----------------+     |          
    | user function:       | <- | user   | <- | user function: | <---+          
    | energy_analysis()    |    | config |    | energy_init()  |                
    +----------------------+    +--------+    +----------------+                
         |             |                                                        
         v             v                                                        
    +----------+   +-------+                                                    
    | Waveform |   | Event | <== The results are then forwarded                 
    +----------+   +-------+     to the rest of the DAQ                         
                                 They may be discarded by the                   
                                 user functions, to eliminate                   
                                 unwanted events.                               

Analysis functions
------------------

Two analysis functions are foreseen in the structure:

1. ``timestamp_analysis()`` that determines the timestamp information of the waveform;
2. ``energy_analysis()`` that determines the energy information of the waveform.

The two analyses were separated in order to simplify code reuse.
The user might want to perform both analyses in the ``energy_analysis()`` function, in order to optimize the computational time.
The functions may be different for each acquisition channel, allowing the coexistence of different detectors on the same digitizer.
A waveform may also be discarded by the user functions, in order to eliminate unwanted/noise events.

Configuration
-------------

The configuration file is in the `JSON <http://www.json.org/>`_ format.
For each channel the user shall provide the filename of the libraries.
The timestamp analysis may be omitted without putting a name on the relative library.
The libraries may be modified and reloaded at runtime.
A reconfiguration command always implies a reaload of the user libraries.
Configurations may be sent through the web-interface or using the python script `send_configuration.py <https://github.com/ec-jrc/abcd/blob/main/waan/send_configuration.py>`_.
The script can send a configuration file stored on disk to a running instance of ``waan``.

.. warning::
    The filenames in the configuration shall follow the conventions of the UNIX dynamic linking functions.
    If the filename contains a slash (``/``) then the file is searched on the given path, if there is no slash then the library is searched in the standard system library folders.
    If the user wants to use a local file in the current directory the filename shall start with ``./`` (_e.g._ ``./libCFD.so``).

Each channel configuration may contain a user defined configuration.
This information is then passed to the ``init`` functions of the user libraries, so the user does not have to hardcode the functions parameters.

Libraries compilation
---------------------

The user libraries shall be implemented with a C99 interface, so no C++.
The ``waan/src/`` folder contains some documented examples.
The user may compile the custom library with the provided Makefile::

    user-tutorial@abcd-tutorial:~/abcd/waan$ make src/libuser.so

Where the source code of the library would be ``src/libuser.c``.

Example libraries
-----------------

These example libraries are provided:

* ``libSimplePSD.c``: Calculates the energy and Pulse Shape information of a short pulse, by applying the double integration method. This is the simplest of the libraries, that can be a starting point for new users.
* ``libLE.c``: Calculates the timing information of a pulse by applying a Leading Edge Discriminator algorithm.
* ``libCFD.c``: Calculates the timing information of a pulse by applying a Constant Fraction Discriminator algorithm.
* ``libRT.c``: Calculates the timing information of a pulse by looking at a threshold crossing point, where the threshold is relative to the pulse maximum. This shows an example of an algorithm that can generate multiple processed events.
* ``libPSD.c``: Calculates the energy and Pulse Shape information of a short pulse, by applying the double integration method.
* ``libStpAvg.c``: Calculates the energy information of a exponentially decaying pulse, by compensating the decay and determining its height with simple averages.
* ``libCRRC4.c``: Calculates the energy information of a exponentially decaying pulse, by compensating the decay and then applying a recursive CR-RC^4 filter.
* ``libRC4.c``: Calculates the energy information of a short pulse, by applying a recursive RC^4 filter and determining the maximum.

User interface
--------------

.. figure:: images/ABCD_waan.png
    :name: fig-ABCD-waan
    :width: 50%
    :alt: interface of the waan module

    Web-interface of the general purpose waveforms analysis module ``waan``.

:numref:`fig-ABCD-waan` shows the web-interface of ``waan``.
On top it shows the measured rates of the enabled acquisition channels.
The rates are shown after the filtering of the user libraries, so the user can immediately see the effect on the rate in case of filtering.
If there are channels in the incoming datastream that are not enabled in ``waan`` they are shown as *channels that are not analyzed*.
The text editor shows the current configuration by clicking on *Get configuration*.
It is possible to modify the ``waan`` configuration by editing the text and clicking on *Send configuration*.
The currently running configuration of ``waan`` is also saved in the raw files.

.. warning::
    Updated configurations that are sent to ``waan`` from the web-interface are never stored on disk.
    The user should manually download the configuration from the web-interface, otherwise the changes will be lost (unless a raw file is opened).