=====================
Interacting with ABCD
=====================

Web-based interface
-------------------

ABCD is meant to run in the background: once started with a startup script nothing will appear on the screen.
The user interface is web-based and thus the user should use a web-browser to connect to it.
With the default setting the interface can be accessed with the link: http://127.0.0.1:8080

.. figure:: images/ABCD_landing_page.png
    :name: fig-ABCD-landing-page
    :width: 50%
    :alt: landing page of the ABCD web-based user interface

    Landing page of the ABCD web-based user interface.

When connected to the address the browser will show a landing page (see :numref:`fig-ABCD-landing-page`).
All the module pages are linked from the landing page.
It also shows the hostname in which the ABCD instance is running, to reduce confusion in case of multiple computers running ABCD.

.. note::
    To navigate between module pages, we suggest to open them in new browser tabs, instead of clicking on their links to change the page.
    When a module page is opened for the first time, it takes several seconds to load all the information from the module.
    If the page was already loaded, the switching between modules' interfaces is faster and more pleasant.

.. _sec-interface-command-line:

Command line interface
----------------------

`tmux <http://tmux.github.io/>`_ is employed in order to run the ABCD instance in the background.
tmux can create virtual terminals that are not attached to a particular graphical window or terminal.
The processes can run in a background session of tmux, but the user can still log into that session, check the running status, and then detach from the session without ever stopping the processes.
In the startup script the system is started in a tmux session called ``ABCD``.
It is possible to attach to the running tmux session from the user running the ABCD instance by::

    user-tutorial@abcd-tutorial:~$ tmux a -t ABCD

It will open up the tmux instance with the list of opened windows on the bottom (see :numref:`attached-tmux-session`).
To detach from the tmux session and leave the ABCD instance running, use ``ctrl-b`` followed by a ``d``.
Refer to the `tmux documentation <https://github.com/tmux/tmux/wiki/Getting-Started>`_ to know how to interact further with the tmux session.

.. code-block:: none
    :name: attached-tmux-session
    :caption: Diagram of an attached tmux session running an instance of ABCD. Normally the first windows is a new bash instance, the ABCD modules are running on the other windows showed on the bottom of the session.

    +-----------------------------------------------------------------------------------+
    | user-tutorial@abcd-tutorial:~/abcd$                                               |
    |                                     ^                                             |
    |                                     |                                             |
    |       +----------------------------------------------------+                      |
    |       | Window 0 is normally empty in the example startups |                      |
    |       +----------------------------------------------------+                      |
    |        |                                                                          |
    |        |      +-----------------------------------------------+                   |
    |        |      | The other windows contain the running modules |                   |
    |        |      +-----------------------------------------------+                   |
    |        |       |      |          |         |                                      |
    |        V       V      V          V         V                                      |
    |                                                                                   |
    | [ABCD] 0:bash* 1:wit  2:loggers  3:replay  4:waan>"abcd-tutorial" 11:07 03-Aug-22 |
    +-----------------------------------------------------------------------------------+

There are also utilities in the ``bin/`` folder that give the possibility to interact from the command line:

* `save_to_file.py <https://github.com/ec-jrc/abcd/blob/main/bin/save_to_file.py>`_: to open a file in which data will be saved. There is the possibility of selecting which file types are to be opened.
* `close_file.py <https://github.com/ec-jrc/abcd/blob/main/bin/close_file.py>`_: to close the currently open file in which data in being saved.
* `send_command.py <https://github.com/ec-jrc/abcd/blob/main/bin/send_command.py>`_: to send a command to a module that support commands reception.
* `stop_ABCD.sh <https://github.com/ec-jrc/abcd/blob/main/startup/stop_ABCD.sh>`_: to stop a currently running ABCD instance (this script is in the ``startup/`` directory).
* `read_events.py <https://github.com/ec-jrc/abcd/blob/main/bin/read_events.py>`_: to read acquisition events from status sockets of modules. Used as a logger (see :ref:`acquisition-logging`).
* `commands_receiver.py <https://github.com/ec-jrc/abcd/blob/main/bin/commands_receiver.py>`_: to create a commands socket to receive commands from other modules. Useful for debugging purposes.
* `read_socket.py <https://github.com/ec-jrc/abcd/blob/main/bin/read_socket.py>`_: to read messages from PUB sockets. Useful for debugging purposes.