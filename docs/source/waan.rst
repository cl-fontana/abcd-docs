.. _ch-waan:

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
A reconfiguration command always implies a reload of the user libraries.
Configurations may be sent through the web-interface or using the python script `send_configuration.py <https://github.com/ec-jrc/abcd/blob/main/waan/send_configuration.py>`_.
The script can send a configuration file stored on disk to a running instance of ``waan``.

.. warning::
    The filenames in the configuration shall follow the conventions of the UNIX dynamic linking functions.
    If the filename contains a slash (``/``) then the file is searched on the given path, if there is no slash then the library is searched in the standard system library folders.
    If the user wants to use a local file in the current directory the filename shall start with ``./`` (*e.g.* ``./libCFD.so``).

Each channel configuration may contain a user defined configuration.
This information is then passed to the ``init`` functions of the user libraries, so the user does not have to hardcode the functions parameters.

Example configuration
---------------------

.. code-block:: JSON
    :name: tab-waan-configuration-example
    :caption: Example configuration of the ``waan`` module. This configuration is used in the example startup that replays example data.

    {
        "module": "waan",
        "forward_waveforms": true,
        "enable_additional": true,
        "discard_messages": false,
        "channels": [
            {
                "id": 1,
                "name": "LaBr",
                "enabled": true,
                "user_libraries": {
                    "timestamp": "src/libCFD.so",
                    "energy": "src/libPSD.so"
                },
                "user_config": {
                    "baseline_samples": 64,
                    "smooth_samples": 16,
                    "fraction": 0.75,
                    "delay": 5,
                    "zero_crossing_samples": 2,
                    "fractional_bits": 10,
                    "disable_shift": true,
                    "pregate": 40,
                    "short_gate": 30,
                    "long_gate": 90,
                    "pulse_polarity": "negative",
                    "integrals_scaling": 2
                }
            },
            {
                "id": [ 6, 7 ],
                "name": "CeBr",
                "enabled": true,
                "user_libraries": {
                    "timestamp": "src/libCFD.so",
                    "energy": "src/libPSD.so"
                },
                "user_config": {
                    "baseline_samples": 64,
                    "smooth_samples": 16,
                    "fraction": 0.75,
                    "delay": 20,
                    "zero_crossing_samples": 2,
                    "fractional_bits": 10,
                    "disable_shift": true,
                    "pregate": 40,
                    "short_gate": 30,
                    "long_gate": 90,
                    "pulse_polarity": "negative",
                    "integrals_scaling": 2
                }
            }
        ]
    }

:numref:`tab-waan-configuration-example` shows a configuration example.
More examples can be found in the ``waan/configs/`` folder.
A detailed list of configurations follows:

* ``forward_waveforms``: Bool value that enables the forwarding of the processed waveforms.
  A user may disable the forwarding in order to avoid the data saver to save them in the raw files, thus reducing their dimensions.
* ``enable_additional``: Bool value the enables the forwarding of the additional waveforms.
  These waveforms are calculated by the user libraries for debugging purposes and in general do not need to be stored.
  In case a user wants to store waveforms, the additional waveforms take up a lot of space and are preferably disabled.
* ``discard_messages``: Bool value the enables the message dropping.
  In case of experiments with very high data throughput the operating system kernel may decide to drop messages to preserve the memory usage.
  This setting should disable that, but it depends on the operating system and it might not actually work.
* ``channels``: Array value of objects.
  This array contains the settings of the single channels.
  Each channel object has the settings:

  - ``id``: Integer value that indicates the channel to which these settings apply.
    It may be substituted with an array of integer values, indicating that these settings are to be replicated to all these channels.
  - ``name``: Just a mnemonic string for the user. The program actually ignores this setting.
  - ``enabled``: Bool value to quickly enable or disable this channel settings.
  - ``user_libraries``: An object containing the paths of the user libraries.
    The ``timestamp`` library is optional, if not set then ``waan`` will use the digitizer timestamp for the waveform.
    The ``energy`` library is mandatory.
    If ``waan`` is unable to load the ``energy`` library then the channels in the ``id`` settings will be disabled.
  - ``user_config``: An object that is provided to the user libraries by ``waan``.
    Its content depend totally on the user libraries. Refer to their documentation.

Libraries compilation
---------------------

The user libraries shall be implemented with a C99 interface, so no C++.
The ``waan/src/`` folder contains some documented examples.
The user may compile the custom library with the provided Makefile::

    user-tutorial@abcd-tutorial:~/abcd/waan$ make src/libuser.so

Where the source code of the library would be ``src/libuser.c``.

Example libraries
-----------------

Some example libraries are provided in the ``waan/src/`` directory.
They are extensively commented and documented in the source code.
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

The tutorial has an extensive description of the web-based user interface (see :numref:`sec-waveforms-analysis-page`).