.. _first-replay:

==============================================
Getting acquainted with a simulated experiment
==============================================

If ABCD was correctly installed (see :ref:`installation`) we can run a simulated experiment to get acquainted with the web-based interface.
ABCD supports the replaying of saved data files (for more information see :ref:`replay`).
Replaying a raw file allows to simulate a full working system.
The replay will display:

* The digitizer configuration;
* Relevant acquisition events (*e.g.* start, stop, reconfigurations, errors,...);
* Acquisition status during the replay (*e.g.* the measured acquisition rates);
* The configuration of the on-line waveforms processing (if it was used during the experiment);
* The acquired waveforms (if saved in the file) and processed events;

There is no need to have the hardware connected to the computer, if the user wants to re-run analyses on old data files or wants to debug analysis functions.
The simulation may also be used as a teaching aid for new users.
ABCD is delivered with some `example data files <https://github.com/ec-jrc/abcd/tree/main/data>`_ that can be used to get started.

Launching a replay
------------------

Some startup scripts are provided for a quick start (for more information see :ref:`startup`).