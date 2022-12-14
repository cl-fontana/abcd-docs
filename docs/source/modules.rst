=================
Available modules
=================

Being ABCD a collection of intependent processes, there is a number of available modules.
All the programs are to be run in the terminal.
Modules have an in-line help that is shown calling the program with the ``-h`` flag.

Hardware interfacing modules
----------------------------

These modules are implemented following the finite state machines paradigm, in order to simplify the development phase.
Diagrams of the state machines can be found in the ``doc/`` folder in the repository.

* `abcd <https://github.com/ec-jrc/abcd/tree/main/abcd/>`_: Data acquisition module, that interfaces with CAEN digitizers.
* `abad2 <https://github.com/ec-jrc/abcd/tree/main/abad2/>`_: Data acquisition module, that interfaces with the Digilent Analog Discovery 2.
* `abrp <https://github.com/ec-jrc/abcd/tree/main/abrp/>`_: Data acquisition module, that interfaces with the Red Pitaya.
* `absp <https://github.com/ec-jrc/abcd/tree/main/absp/>`_: Data acquisition module, that interfaces with SP Devices digitizers.
* `abps5000a <https://github.com/ec-jrc/abcd/tree/main/abps5000a/>`_: Data acquisition module, that interfaces with the PicoScope 5000A.
* `hivo <https://github.com/ec-jrc/abcd/tree/main/hivo/>`_: High-voltages management module, that interfaces with CAEN VME high-voltage power supplies.

On-line analysis modules
------------------------

* `spec <https://github.com/ec-jrc/abcd/tree/main/spec/>`_: Histogramming module, that calculates the energy histograms on-line.
* `tofcalc <https://github.com/ec-jrc/abcd/tree/main/tofcalc/>`_: On-line Time of Flight calculation between some reference channels and the others (see :numref:`ch-tofcalc` for more information).
* `waan <https://github.com/ec-jrc/abcd/tree/main/waan/>`_: General purpose waveforms processing module that uses user-supplied pulse processing libraries (see :numref:`ch-waan` for more information).

Datastream filters
------------------

Filters are usually implemented as simple loops reading from the sockets, selecting the processed events and forwarding the data to the next module.

* `chafi <https://github.com/ec-jrc/abcd/tree/main/chafi/>`_: Channel filter, that filters the waveforms and processed events allowing the passage of only the data coming from a specific channel.
* `cofi <https://github.com/ec-jrc/abcd/tree/main/cofi/>`_: Coincidence filter, that filters the processed events allowing the passage of only the data in coincidence in a specified time-window (see :numref:`ch-cofi` for more information).
* `enfi <https://github.com/ec-jrc/abcd/tree/main/enfi/>`_: Energy filter that filters the processed events allowing the passage of only the data with an energy in a specified interval.
* `pufi <https://github.com/ec-jrc/abcd/tree/main/pufi/>`_: Pulse Shape Discrimination filter, that selects data that fall in a polygon on the (energy, PSD) plane.

Utilities modules for the framework
-----------------------------------

* `wit <https://github.com/ec-jrc/abcd/tree/main/wit/>`_: Web-based user interface module, that hosts the web-service.
* `wadi <https://github.com/ec-jrc/abcd/tree/main/wadi/>`_: Waveforms display, that reads the waveforms data streams and formats it for visualization purposes for the web-based interface. It can only be used in conjunction with ``wit``.
* `dasa <https://github.com/ec-jrc/abcd/tree/main/dasa/>`_: Data saving module, that saves data to disk.
* `fifo <https://github.com/ec-jrc/abcd/tree/main/fifo/>`_: Temporary data storage with a FIFO structure, data are kept in memory for a given time span and then are discarded.
* `gzad <https://github.com/ec-jrc/abcd/tree/main/gzad/>`_: Compress data, filter that compresses with the zlib or the bz2 libraries the messages in a socket stream (compression ratio about 50%).
* `unzad <https://github.com/ec-jrc/abcd/tree/main/gzad/>`_: Decompress data, filter that decompresses the packets generated by the ``gzad`` filter.