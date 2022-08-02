====================
Introduction on ABCD
====================

ABCD is a *distributed* Data Acquisition (DAQ) framework, with the aim of acquiring data from signal digitizers in Nuclear Physics experiments.
It is distributed in the sense that each task related to the DAQ runs in a separate process.
Typical tasks are the reading of data from signal digitizers, on-line processing of waveforms, or data storing.
ABCD is therefore a framework of independent processes, also called *modules*.
The exchange of information between the processes is obtained through dedicated `communication sockets <https://en.wikipedia.org/wiki/Network_socket>`_.
The low-level implementation details of the sockets are abstracted by the excellent `ZeroMQ messaging library <https://zeromq.org/>`_.
ABCD supports the acquisition from various signal digitizers from different vendors (*e.g.* `CAEN <http://www.caen.it/>`_, `SP Devices <https://www.spdevices.com/>`_, `Red Pitaya <https://www.redpitaya.com/>`_, and `Digilent <https://store.digilentinc.com/>`_).

Distributed systems
-------------------

A distributed system has a very versatile structure that may be changed on-the-fly, by changing the connections between modules.
Parallel and distributed programming are inherent between the various processes.
The computational load may be easily shared over the network to different computers.
Data is distributed over streams and data processors may be dynamically connected.
Several processes may run different analyses in parallel on-line on the same data, for early testing phases. 

The processes are isolated by the communication sockets and can exchange only the relevant information.
This isolation simplifies the collaboration between developers, as independently developed processes may be attached to the network and heterogeneous programming languages can coexist.
The communication protocols are based on `JSON <https://www.json.org/>`_ messages for high-level information and on standardized binary protocols for high performance data.

A downside of a distributed system is the complexity of synchronizing the independent processes.
On the ABCD case, though, this has not been a problem so far.
The data processing normally follows pipelines with sequential steps and the data goes only downstream.
Therefore it is not necessary to push data upstream to synchronized modules.

Modules
-------

The modules are implemented to be simple and lightweight.
Their simplicity eases their development and thus their debugging and assessment.
Each simple process tends to be also more reliable than a huge monolithic combination of modules.
Being lightweight they may be run in resource constraint systems, such as embedded computers, but also in data intensive experiments.

Normally modules use the communication sockets:

* Status socket, implemented as a `ZeroMQ PUB <https://zguide.zeromq.org/docs/chapter1/#Getting-the-Message-Out>`_, that produces JSON messages containing information on the running status and significant events;
* Commands socket, implemented as a `ZeroMQ PULL <https://zguide.zeromq.org/docs/chapter1/#Divide-and-Conquer>`_, that receives JSON messages allowing other processes to control this process;
* Data socket, implemented as a ZeroMQ PUB, that produces binary messages containing the data acquired by signal digitizers and the corresponding processes data.

A module may read messages from one or multiple modules on these sockets and send command messages to other modules.

Typical system structure
------------------------

.. code-block:: none
    :name: diagram-simple-connections
    :caption: Simplified diagram of typical connections between modules for a simple experiment with on-line data processing.

                         +----------------+
                         |   Web-based    |
                         | user-interface |
                         +----------------+
                               ^     | Commands
                        Status !     | sockets
                        socket !     +------------+
                               !     |            |
                    Data       !     V            |
    +-----------+  stream  +-------------+        |
    |           |=========>|  Hardware   |        |
    | Digitizer |          | interfacing |        |
    |           |<---------|   module    |        |
    +-----------+ Commands |             |        |
                           +-------------+        |
                               :                  |
                          Data :     +------------+
                        socket :     |            |
                               V     V            |
                           +------------+         |
                           |  On-line   |         |
                           | waveforms  |         |
                           | processing |         |
                           +------------+         |
                               :                  |
                          Data :                  |
                        socket :..............    |
                               :             :    |
                               V             V    V
                           +-------------+ +--------+
                           |   On-line   | |  Data  |
                           |   spectra   | | saving |
                           | calculation | | module |
                           +-------------+ +--------+

:numref:`diagram-simple-connections` shows a simplified diagram of typical connections between modules for a simple experiment.
A digitizer is connected to a module that interfaces the ABCD framework with the hardware.
The interfacing module translates the digitizer data and status to the ABCD format, for uniform data treatment.
The waveforms data that is produced is then analyzed by an on-line waveforms processing module.
The processed data (*e.g.* pulse heights of the waveforms) is fed to a module calculating on-line the energy spectra.
A copy of the processed datastream is fed to a module that saves it to a permanent storage.
The whole structure is controlled by the web-based user interface.

If another digitizers is to be used, it is sufficient to change the hardware interfacing module and the rest of the connections are unmodified.
The standardized data format ensures the intercomunication between the modules.