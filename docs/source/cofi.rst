.. _ch-cofi:

============================================
Temporal coincidence filter: ``cofi`` module
============================================

``cofi`` filters the processed events allowing the passage of only the data in coincidence with a reference channel in a specified time-window.

Algorithm
---------

The ``cofi`` module works on data packets generating groups of events in coincidence with the *reference channels*.
Reference channels are selected by the user as the channels giving the reference timestamp for the time differences calculations.
This algorithm is the similar to the one employed by the ``tofcalc`` module (see :numref:`ch-tofcalc`).
The applied algorithm is:

1. **Sort** all the events in a packet according to the timestamps.
2. **Select** the events generated by *reference channels*.
   For each reference channel event then:

   1. **Search backward** in the events array for events that did not originate from a reference channel.
      If the timestamps are outside the left time window stop the search.
      Store the selected events in the output buffer.
   2. **Search forward** for events that did not originate from a reference channel.
      If the timestamps are outside the right time window, stop the search.
      Store the selected events in the output buffer.

Coincidence windows
-------------------

.. code-block:: none
    :name: tab-diagram-coincidence-windows-cofi
    :caption: Diagram of the coincidence windows

                  Events from           Event from a
                  other channels        reference channel
                  in the left           |
                  coincidence           |
                  window                |
                  |                     |
                  +----------+-----+    |
                  |          |     |    |
                  V          V     V    V
    
      3           2          2     1    0         1   1      3    2
    ------------------------------------|----------------------------> timestamps
              |                         t0 <- Time zero of the     |
              |                         |     coincidence window   |
              |-------------------------|--------------------------|
              | Left coincidence window   Right coincidence window |
              |                                                    |
              |                                                    |
             -t_l <- Left edge                       Right edge -> t_r
                     of the window                of the window

:numref:`tab-diagram-coincidence-windows-cofi` shows a diagram of the coincidence windows defined by the module.

Output
------

The data format is the same as the rest of ABCD, but the events are grouped in coincidence groups.
The **first event** in a group is always an event from a reference channel.
The first event contains in the ``group_counter`` entry the number of events that are part of this group.
The following ``group_counter`` events should be treated as part of the coincidence group.

.. table:: Example output of the grouped events from the ``cofi`` module. For this example the reference channel is number 1.
    :name: tab-cofi-example-output

    ============  ======  =====  =======  =======  ==============================
    timestamp     qshort  qlong  channel  group
                                          counter
    ============  ======  =====  =======  =======  ==============================
    112819094756  7657    24960  1        **1**    Reference event in group
    112819133997  2539    8275   7        0        Associated event
    140943273296  7320    20021  1        **0**    Ref. ev. in empty group
    159776923787  18738   59958  1        **2**    Ref. ev. in new group
    159776887430  2705    8789   7        0        Associated event
    159776894675  1112    3516   6        0        Associated event
    690314730482  5551    19317  1        **1**    Ref. ev. in new group
    690314697733  1787    5404   7        0        Associated event
    ============  ======  =====  =======  =======  ==============================


Program options
---------------

This module runs only in the command line and has no associated graphical user interface.
The program expect a list of the reference channels in the command line::
    
    user-tutorial@abcd-tutorial:~/abcd/cofi$ ./cofi [options] <reference_channels>

The optional arguments are:

- ``-h``: Display the inline help that also shows the default values
- ``-v``: Set verbose execution
- ``-V``: Set verbose execution with more output
- ``-A <address>``: Input socket address
- ``-D <address>``: Output socket address
- ``-T <period>``: Set base period in milliseconds
- ``-w <coincidence_window>``: Use a symmetric coincidence window. Both the left
  as well as the right coincidence windows will have a *width* of
  ``coincidence_window`` nanoseconds.
- ``-l <left_coincidence_window>``: Left coincidence window width in nanoseconds.  This is the width of the left coincidence window (*i.e.* in general it is a positive number).
- ``-r <right_coincidence_window>``: Right coincidence window width in nanoseconds.
  This is the width of the right coincidence window.
- ``-L <left_coincidence_edge>``: Left edge of the left coincidence window in
  nanoseconds. This is the time value of the left edge of the left coincidence
  window (*i.e.* in general it is a negative value equal to
  ``-1 * left_coincidence_window``).
- ``-m <multiplicity>``: Event minimum multiplicity, excluding the reference
  channel. The minimum number of events in a coincidence groups that enable the
  forwarding of this group.
- ``-n <ns_per_sample>``: Conversion factor of nanoseconds per timestamp sample.
- ``-k``: Enable keep reference event, even if there are no coincidences.
  The reference events will be always forwarded but not the other events.