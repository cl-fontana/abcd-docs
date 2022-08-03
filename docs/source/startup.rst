.. _startup:

=======================
Starting up an instance
=======================

The `startup directory <https://github.com/ec-jrc/abcd/tree/main/startup>`_ contains some example scripts for starting up an ABCD instance.
It is also possible to run an ABCD instance at the startup of the computer (see: :ref:`service`).
New users are suggested to start with `startup_example_replay.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_replay.sh>`_, so they can see a working system with the example data.

.. warning::
    All the scripts require to know where ABCD is installed.
    Modify, in the scripts, the variable ``$ABCD_FOLDER`` accordingly, or define it as an environment variable in the shell.

.. note::
    ABCD is meant to run in the background: once started with a startup script nothing will appear on the screen.
    The user interface is web-based and thus the user should use a web-browser to connect to the address: http://127.0.0.1:8080
    The default port for the web-interface is 8080, but it can be changed in the configuration.

Scripts description
-------------------

* `startup_example_replay.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_replay.sh>`_ shows a standard startup for a replay of previously acquired data. The script uses the example data in the `data/ folder <https://github.com/ec-jrc/abcd/tree/main/data>`_.  ``waan`` is used to process the waveforms. Optionally, the script may accept a data file as argument::

    user-tutorial@abcd-tutorial:~/abcd/startup$ ./startup_example_replay.sh ~/abcd/data/example_data_DT5730_Ch2_LaBr3_Ch4_LYSO_Ch6_YAP_raw.adr.bz2

* `startup_example.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example.sh>`_ shows a standard startup for a digitizer that processes waveforms on board.
* `startup_example_AD2.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_AD2.sh>`_ shows a standard startup for a digitizer that can only acquire waveforms (*i.e.* no processing is done by the digitizer). ``waan`` then processes the waveforms on the computer, obtaining the energy information. Finally, ``dasa`` can save the processed datastream.
* `startup_example_waan.sh <https://github.com/ec-jrc/abcd/blob/main/startup/startup_example_waan.sh>`_ shows a standard startup for a digitizer that can only acquire waveforms. ``waan`` then processes the waveforms on the computer, obtaining the energy information. On the other hand, ``dasa`` is attached to the original datastream and will only save waveforms.
* `C-BORD_startup.sh <https://github.com/ec-jrc/abcd/blob/main/startup/C-BORD_startup.sh>`_ shows a rather complex startup with two digitizers being acquired in parallel.
* `stop_ABCD.sh <https://github.com/ec-jrc/abcd/blob/main/startup/stop_ABCD.sh>`_ is a script to stop a running instance of ABCD.

.. _check-running:

Checking if ABCD is running
---------------------------

ABCD runs in the background and it is not directly evident if it is running.
There are a few steps to check if it is running:

#. tmux hosts the ABCD instance while it is running in the background. Check if there is a tmux instance running in the background::

        user-tutorial@abcd-tutorial:~/abcd$ tmux ls
        ABCD: 9 windows (created Tue Aug  2 16:47:07 2022)

   It should show an instance called ABCD with **several** windows open. If there is only one window open or it shows::

        user-tutorial@abcd-tutorial:~/abcd$ tmux ls
        no server running on /tmp/tmux-1000/default

   then the startup failed. Was ABCD correctly installed and compiled? See :ref:`installation` and :ref:`manual-installation-compilation`.
   Was the ``$ABCD_FOLDER`` correctly set? See warning in :ref:`startup`

#. The default setting of the web-based interface is to open port 8080, thus connect to: http://127.0.0.1:8080/
   If the browser shows the landing page of ABCD then the web-based interface is probably working.
   If the browser is unable to connect to the page or cannot reach the page, then the web interface is not running.
   Were the dependencies fo the web interface correctly installed? See :ref:`manual-installation-wit`.

#. Check at the bottom of the modules pages the connection status of the modules.
   If one of the modules is not running, try to run it directly in a terminal window. The module can show error messages to indicate the problem.
   To directly run a module in the terminal copy the launch string from the startup script.

.. _acquisition-logging:

Acquisition events logging
--------------------------

The example startup scripts activate some events loggers, that save all the significant events relative to an acquisition (*e.g.* start, stop, reconfigurations,...).
The loggers are `python scripts <https://github.com/ec-jrc/abcd/blob/main/bin/read_events.py>`_ that read the status sockets and log the events.
The logs are saved as text files in the ``log/`` directory of the ABCD main folder.

.. _service:

Automatic startup of ABCD at computer startup
---------------------------------------------

ABCD is designed to be always running in the computer and be remotely controlled.
It is possible to run a startup script at the boot of the computer, without having to log in as a user.
For more information about startup scripts see: :ref:`startup`.

systemd service installation
````````````````````````````

A service file is provided in the repository with the settings to run an ABCD instance automatically at the computer boot.
This service file works only on Linux distributions using `systemd <https://systemd.io/>`_.
To install the service run the `installation script <https://github.com/ec-jrc/abcd/blob/main/service/install_service.sh>`_::

    user-tutorial@abcd-tutorial:~/abcd/startup$ ./install_service.sh -S ../startup/startup_example_replay.sh 
    Setting startup_script to: ../startup/startup_example_replay.sh
    Startup script with full path: /home/user-tutorial/abcd/startup/startup_example_replay.sh
    Username: user-tutorial                                                                                        

the installation script accepts some options that can be selected to customize the installation.
A help message is available calling the script with::

    user-tutorial@abcd-tutorial:~/abcd/startup$ ./install_service.sh -h

To check if the service is successfully started::

    user-tutorial@abcd-tutorial:~/abcd/startup$ journalctl -xeu abcd.service                                                                                                                          
    Aug 03 10:02:12 abcd-tutorial systemd[1]: Starting ABCD data acquisition...                              
    Subject: A start job for unit abcd.service has begun execution
    Defined-By: systemd
    Support: http://www.ubuntu.com/support
                                                                                                                                                                                                                    
    A start job for unit abcd.service has begun execution.                                               
                                
    The job identifier is 18343.
    Aug 03 10:02:12 abcd-tutorial bash[176359]: Today is 20220803                                            
    Aug 03 10:02:12 abcd-tutorial bash[176359]: Replaying data file: /home/user-tutorial/abcd//data/example_data_DT5730_Ch1_LaBr3_Ch6_CeBr3_Ch7_CeBr3_coincidence_raw.adr.bz2                                                
    Aug 03 10:02:12 abcd-tutorial bash[176359]: Starting a new ABCD session
    Aug 03 10:02:12 abcd-tutorial bash[176359]: Creating the window for the GUI webserver: WebInterfaceTwo   
    Aug 03 10:02:12 abcd-tutorial bash[176368]: ABCD:1.0             
    Aug 03 10:02:12 abcd-tutorial bash[176359]: Creating loggers window                    
    Aug 03 10:02:12 abcd-tutorial bash[176372]: ABCD:2.0                                                     
    Aug 03 10:02:12 abcd-tutorial bash[176375]: ABCD:2.1                                                     
    Aug 03 10:02:12 abcd-tutorial bash[176378]: ABCD:2.1                                                     
    Aug 03 10:02:12 abcd-tutorial bash[176382]: ABCD:2.1                                                     
    Aug 03 10:02:12 abcd-tutorial bash[176359]: Waiting for node.js to start        
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating replayer window, file: /home/user-tutorial/abcd//data/example_data_DT5730_Ch1_LaBr3_Ch6_CeBr3_Ch7_CeBr3_coincidence_raw.adr.bz2                                     
    Aug 03 10:02:14 abcd-tutorial bash[176410]: ABCD:3.0    
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating WaAn window                                         
    Aug 03 10:02:14 abcd-tutorial bash[176413]: ABCD:4.0                                                     
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating DaSa window, folder: /home/user-tutorial/abcd//data/      
    Aug 03 10:02:14 abcd-tutorial bash[176416]: ABCD:5.0             
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating WaDi window                                         
    Aug 03 10:02:14 abcd-tutorial bash[176419]: ABCD:6.0        
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating tofcalc windows         
    Aug 03 10:02:14 abcd-tutorial bash[176422]: ABCD:7.0                             
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating spec windows                   
    Aug 03 10:02:14 abcd-tutorial bash[176427]: ABCD:8.0                                                                                                                                                               
    Aug 03 10:02:14 abcd-tutorial bash[176359]: System started!    
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Connect to GUI on addresses: http://127.0.0.1:8080/
    Aug 03 10:02:14 abcd-tutorial systemd[1]: Finished ABCD data acquisition.                                
    Subject: A start job for unit abcd.service has finished successfully                                                                                                                                            
    Defined-By: systemd                                                                                                                                                                                             
    Support: http://www.ubuntu.com/support                                                                
                                                                                                          
    A start job for unit abcd.service has finished successfully.                                          
                                                                                                                                                                                                                    
    The job identifier is 18343.                                       