.. _ch-startup:

=======================
Starting up an instance
=======================

The startup directory ``/usr/share/abcd/startup/`` contains some example scripts for starting up an ABCD instance.
It is also possible to run an ABCD instance at the startup of the computer (see: :numref:`sec-service`).
New users are suggested to start to look at ``startup_example_hardware.sh``, so they can see a simple startup for some digitizer.
The startup ``startup_example_replay.sh`` is somewhat more complex, as it used to replay some data files (see: :numref:`ch-first-replay`).

.. note::
    ABCD is meant to run in the background: once started with a startup script nothing will appear on the screen.
    The user interface is web-based and thus the user should use a web-browser to connect to the address: http://127.0.0.1:8080
    The default port for the web-interface is 8080, but it can be changed in the configuration.
    If ABCD is running inside the `Windows Subsystem for Linux 2 (WSL2) <https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux>`_ then the address for the web-interface might be different.
    Check https://learn.microsoft.com/en-us/windows/wsl/networking for more information.

.. _sec-startup-description:

Scripts description
-------------------

* ``startup_example_replay.sh`` shows a standard startup for a replay of previously acquired data.
  The script uses the example data in the ``/usr/share/abcd/data/`` directory.
  ``waan`` is used to process the waveforms. Optionally, the script may accept a data file as argument::

    user-tutorial@abcd-tutorial:~$ /usr/share/abcd/startup/startup_example_replay.sh /usr/share/abcd/data/example_data_DT5730_Ch2_LaBr3_Ch4_LYSO_Ch6_YAP_raw.adr.bz2

* ``startup_example_CAEN.sh`` shows a startup for a digitizer that processes waveforms on board.
* ``startup_example_hardware.sh`` shows a startup for a digitizer that can only acquire waveforms (*i.e.* no processing is done by the digitizer). ``waan`` then processes the waveforms on the computer, obtaining the energy information.
* ``stop_ABCD.sh`` is a script to stop a running instance of ABCD.

Shutting down ABCD
------------------

Use the script: ``stop_ABCD.sh``.

.. _sec-check-running:

Checking if ABCD is running and troubleshooting
-----------------------------------------------

ABCD runs in the background and it is not directly evident if it is running.
There are a few steps to check if it is running:

#. tmux hosts the ABCD instance while it is running in the background. Check if there is a tmux instance running in the background::

        user-tutorial@abcd-tutorial:~$ tmux ls
        ABCD: 9 windows (created Tue Aug  2 16:47:07 2022)

   It should show an instance called ABCD with **several** windows open. If there is only one window open or it shows::

        user-tutorial@abcd-tutorial:~$ tmux ls
        no server running on /tmp/tmux-1000/default

   then the startup failed. Was ABCD correctly installed and compiled? See :numref:`installation` and :numref:`manual-installation-compilation`.

#. The default setting of the web-based interface is to open port 8080, thus connect to: http://127.0.0.1:8080/
   If the browser shows the landing page of ABCD then the web-based interface is probably working.
   If the browser is unable to connect to the page or cannot reach the page, then the web interface is probably not running.
   Things to check:

   - Is there a firewall blocking the connections on that port?
   - Is there another process running on the computer using that port?
     The port 8080 is commonly used by local web-servers of various programs, for instance LabView uses that port.
     Change the HTTP port in the startup script adding a ``-p`` option to the call of the web-server::

        user-tutorial@abcd-tutorial:~$ cd /usr/libexec/abcd/wit/
        user-tutorial@abcd-tutorial:~$ node app.js -p 9090 config.json

   - Is ABCD installed in WSL2? 
     The address for the web-interface might be different.
     Check https://learn.microsoft.com/en-us/windows/wsl/networking for more information.

#. Check at the bottom of the modules pages the connection status of the modules.
   If one of the modules is not running, try to run it directly in a terminal window. The module can show error messages to indicate the problem.
   To directly run a module in the terminal copy the launch string from the startup script.

.. _sec-acquisition-logging:

Acquisition events logging
--------------------------

The example startup scripts activate an events logger, that save all the significant events relative to an acquisition (*e.g.* start, stop, reconfigurations,...).
The logger is a `python script <https://github.com/ec-jrc/abcd/blob/main/bin/log_status_events.py>`_ that reads the status sockets and log the events.
The logs are saved as TSV files in the ``log/`` directory of the data directory created by the startup.

.. _sec-service:

Automatic startup of ABCD at computer startup
---------------------------------------------

ABCD is designed to be always running in the computer and be remotely controlled.
It is possible to run a startup script at the boot of the computer, without having to log in as a user.
For more information about startup scripts see: :numref:`ch-startup`.

systemd service installation
````````````````````````````

A service file is provided in the repository with the settings to run an ABCD instance automatically at the computer boot.
This service file works only on Linux distributions using `systemd <https://systemd.io/>`_.
To install the service run the `installation script <https://github.com/ec-jrc/abcd/blob/main/service/install_service.sh>`_::

    user-tutorial@abcd-tutorial:~$ ./install_service.sh -S /usr/share/abcd/startup/startup_example_replay.sh 
    Setting startup_script to: /usr/share/abcd/startup/startup_example_replay.sh
    Startup script with full path: /usr/share/abcd/startup/startup_example_replay.sh
    Username: user-tutorial                                                                                        

the installation script accepts some options that can be selected to customize the installation.
A help message is available calling the script with::

    user-tutorial@abcd-tutorial:~$ ./install_service.sh -h

To check if the service is successfully started::

    user-tutorial@abcd-tutorial:~$ journalctl -xeu abcd.service                                                                                                                          
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
    Aug 03 10:02:14 abcd-tutorial bash[176359]: Creating DaSa window, directory: /home/user-tutorial/abcd//data/      
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