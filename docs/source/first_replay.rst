.. _first-replay:

==============================================
Getting acquainted with a simulated experiment
==============================================

If ABCD was correctly installed (see :numref:`installation`) we can run a simulated experiment to get acquainted with the web-based interface.
ABCD supports the replaying of saved data files (for more information see :numref:`replay`).
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

Some startup scripts are provided for a quick start (for more information see :numref:`ch-startup`).
The script `startup_example_replay.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_replay.sh>`_ startups an ABCD instance with a default replay of an example data file.
The same script may be also used to replay any raw file (see :numref:`sec-startup-description`).

The default example data was acquired with a CAEN DT5730 digitizer.
Three detectors were attached:

* one LaBr detector on channel 1;
* two CeBr detectors on channels 6 and 7;

The digitizer was set to acquire only signals in coincidence between the LaBr (ch0) and either one of the CeBr (ch6 & ch7).
The CeBr detectors were detecting the internal radioactivity of the LaBr.

Let us launch the script from the main abcd folder::

    user-tutorial@abcd-tutorial:~/abcd$ ./startup/startup_example_replay.sh
    Today is 20220804
    Replaying data file: /home/user-tutorial/abcd//data/example_data_DT5730_Ch1_LaBr3_Ch6_CeBr3_Ch7_CeBr3_coincidence_raw.adr.bz2
    Kiling previous ABCD sessions
    Starting a new ABCD session
    Creating the window for the GUI webserver: WebInterfaceTwo
    ABCD:1.0
    Creating loggers window
    ABCD:2.0
    ABCD:2.1
    ABCD:2.1
    ABCD:2.1
    Waiting for node.js to start
    Creating replayer window, file: /home/user-tutorial/abcd//data/example_data_DT5730_Ch1_LaBr3_Ch6_CeBr3_Ch7_CeBr3_coincidence_raw.adr.bz2
    ABCD:3.0
    Creating WaAn window
    ABCD:4.0
    Creating DaSa window, folder: /home/user-tutorial/abcd//data/
    ABCD:5.0
    Creating WaDi window
    ABCD:6.0
    Creating tofcalc windows
    ABCD:7.0
    Creating spec windows
    ABCD:8.0
    System started!
    Connect to GUI on addresses: http://127.0.0.1:8080/

The script creates a ``tmux`` session in the background in which the ABCD instance runs (for more information about ``tmux`` see :numref:`sec-interface-command-line`).
Nothing will be displayed on the screen.
To interact with the user-interface connect with a web-browser to the address: http://127.0.0.1:8080/
The user-interface is hosted by a local web-server running on the local computer.
:numref:`fig-ABCD-landing-page-tutorial` shows the landing page that the browser should show; see :numref:`sec-check-running` if the browser cannot show the landing page.

.. figure:: images/ABCD_landing_page.png
    :name: fig-ABCD-landing-page-tutorial
    :width: 50%
    :alt: landing page of the ABCD web-based user interface

    Landing page of the ABCD web-based user interface.
