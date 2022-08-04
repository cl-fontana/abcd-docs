.. _hardware-support:

================
Hardware support
================

Various signal digitizers were tested with ABCD.
There is support both for digitizers that provide only waveforms and for those that analyze on-board the waveforms.
Refer to :numref:`vendors-libraries` for some tips for the installation of the relative drivers.

CAEN digitizers
---------------

CAEN digitizers are interfaced by the ``abcd`` module.
The models that were tested with ABCD are:

* x720, both in VME and desktop versions;
* x725, desktop form factor;
* x730, both in VME and desktop versions;
* x751, desktop version.

All digitizers were tested with the DPP-PSD firmware.

Tested communication interfaces are:

* V1718 for VME-USB communication;
* A2818, A3818, A4818 for optical fiber communication.

SP Devices
----------

SP Devices digitizers are interfaced by the ``absp`` module.
The models that were tested with ABCD are:

* ADQ214, with the standard firmware in the PXI version;
* ADQ412, with the standard firmware in the PXI version;
* ADQ14, with the standard firmware FWDAQ in the USB and PXI versions;
* ADQ14, with an initial support for the FWPD in the PXI version.

Digilent
--------

The Digilent Analog Discovery 2 (AD2) is interfaced by the ``abad2`` module.

Red Pitaya
----------

Only the Red Pitaya's STEMlab 125-14 was tested with ABCD.
The module providing support for the hardware is the ``abrp`` module.

PicoScope
---------

PicoScope oscilloscopes of the series 5000 have an initial support with the ``abps5000a`` module.