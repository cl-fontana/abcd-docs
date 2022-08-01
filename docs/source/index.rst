====================================
Welcome to the ABCD's documentation!
====================================

**ABCD** is a distributed Data Acquisition framework (DAQ), in which each task related to the DAQ runs in a separate process (*e.g.* data acquisition, hardware management, data processing, analysis,...).
Its main application is the acquisition of data from signal digitizers in Nuclear Physics experiments.
The system is composed of a set of simple modules that exchange information through dedicated `communication sockets <https://en.wikipedia.org/wiki/Network_socket>`_.
ABCD supports the acquisition from various signal digitizers from different vendors (*e.g.* `CAEN <http://www.caen.it/>`_, `SP Devices <https://www.spdevices.com/>`_, `Red Pitaya <https://www.redpitaya.com/>`_, and `Digilent <https://store.digilentinc.com/>`_).
The user interface is implemented as a web-service and can be accessed with a regular web browser.

Check :doc:`hardware_support` for further information on the list of tested digitizers, the :doc:`tutorial` for a step-by-step guide that includes the :ref:`installation` instructions.

.. note::

   This project is under active development.

Contents
--------

.. toctree::

   hardware_support
   tutorial