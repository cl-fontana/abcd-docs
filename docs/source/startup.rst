.. _startup:

========================
Startup scripts examples
========================

This folder contains some example scripts for starting up an ABCD instance.
New users are suggested to start with `startup_example_replay.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_replay.sh>`_, so they can see a working system with the example data.

.. warning::
    All the scripts require to know where ABCD is installed.
    Modify in the scripts the variable ``$ABCD_FOLDER`` accordingly, or define it as an environment variable in the shell.

Scripts description
-------------------

* `startup_example_replay.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_replay.sh>`_ shows a standard startup for a replay of previously acquired data. The script uses the example data in the [`data/`](../data/) folder. `waan` is used to process the waveforms. Optionally, the script may accept a data file as argument::

    user-tutorial@abcd-tutorial:~/abcd/startup$ ./startup_example_replay.sh ~/abcd/data/example_data_DT5730_Ch2_LaBr3_Ch4_LYSO_Ch6_YAP_raw.adr.bz2

* `startup_example.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_CAEN.sh>`_ shows a standard startup for a digitizer that processes waveforms on board.
* `startup_example_AD2.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_AD2.sh>`_ shows a standard startup for a digitizer that can only acquire waveforms (_i.e._ no processing is done by the digitizer). `waan` then processes the waveforms on the computer, obtaining the energy information. Finally, `dasa` can save the processed datastream.
* `startup_example_waan.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_waan.sh>`_ shows a standard startup for a digitizer that can only acquire waveforms. `waan` then processes the waveforms on the computer, obtaining the energy information. On the other hand, `dasa` is attached to the original datastream and will only save waveforms.
* `C-BORD_startup.sh <https://github.com/ec-jrc/abcd/blob/main/startup/C-BORD_startup.sh>`_ shows a rather complex startup with two digitizers being acquired in parallel.
* `stop_ABCD.sh <https://github.com/ec-jrc/abcd/blob/main/startup/stop_ABCD.sh>`_ is a script to stop a running instance of ABCD.
