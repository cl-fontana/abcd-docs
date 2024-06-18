.. _ch-timestamps:

========================
Timestamps determination
========================

Introduction
------------

.. code-block:: none
    :name: diagram-timestamp-determination
    :caption: Diagram of the typical determination of the timestamp in a digitizer.

                          ...  <= Waveform signal
                         .   ..
          ADC samples => X     .
                        .       ..
                        .         ..  
                       .            X..  
                       .               ...
                      .                   ..
                     .                      ..X
     Threshold => - -.- - - - - - - - - - - - -...- - - - - - - - - - - - - - -
         level      .|                            ...
                  ..                                 ...X...
      X.........X.   |                                      ......X.........X...
    --+----+----+----+----+----+----+----+----+----+----+----+----+----+----+---
      |    |    |    |    |    |    |    |    |    |    |    |    |    |    |   
      N   N+1  N+2  N+3  N+4 <= Timestamp clock ticks  ...  ...  ...  ...  ...
                     ^
                     Threshold crossing clock tick
    --+---------+---------+---------+---------+---------+---------+---------+---
      |         |         |         |         |         |         |         |
      M        M+1       M+2 <= ADC samples ticks      ...       ...       ...

Most digitizers provide a timestamp, that is the characteristic temporal information associated to a waveform.
Normally it is a value obtained with a digital counter, that provides integer numbers.
This counter counts the number of cycles (ticks) of a digital clock, see diagram :numref:`diagram-timestamp-determination`.
The timestamp clock might be the same clock guiding the ADC sampling or it might be a different clock.
For instance, in CAEN digitizers the timestamp and ADC sampling clocks are the same while in SP Devices digitizers the timestamp clock is faster than the ADC sampling.

Normally the timestamp is determined when the signal crosses a threshold level, so it has the precision of the digital clock determining it.
In diagram :numref:`diagram-timestamp-determination` the timestamp value would be ``N+3``.
The threshold crossing happens between samples number ``M+1`` and ``M+2`` of the ADC clock.

Precision of digital numbers
----------------------------

Being the timestamp an integer, it is an exact number and its precision is the very same as the digital clock used to determine it.
Normally the clocks runs at frequencies of the order of few hundreds of MHz or a few GHz.
Assuming that the clock is 1 GHz, for simplicity, the time step would be of the order of 1 ns.
Thus after 1 s of acquisition time, the value of a timestamp can be of the order of 1 billion (10\ :sup:`9`).
Since in some experiments we want to measure time differences of the order of 10 ns or less, we need a very high precision on the timestamp number.
Floating point values are approximated values and thus they might lose the needed precision.
64 bits floating point values have a precision of `53 bits <https://en.wikipedia.org/wiki/Floating-point_arithmetic#Range_of_floating-point_numbers>`_.
The minimum difference between two floating point values depends also on their magnitude.
Integers, on the other hand, have a fixed precision that does not depend on their magnitude.
For these reasons, most of the timestamps are stored as 64 bits unsigned integers.

Fixed-point values
------------------

.. code-block:: none
    :name: diagram-fixed-point
    :caption: Diagram of fixed-point numbers.

                                     Integer number: T = 3201942
     
                   Equivalent floating point number: T = 3.201942e6
     
                      Equivalent fixed-point number: T = 3201942.000
        (with three digits of fractional precision)
     
            Emulating fixed-point fractional number: T · 1000 = 3201942000

The best precision that an integer number can provide is one clock sample, thus we might want to improve that precision (especially for the slower digitizers).
There is the possibility of obtaining a better precision of the timestamps by interpolating the ADC samples.
Referring to diagram :numref:`diagram-timestamp-determination`, we can interpolate the samples number ``M+1`` and ``M+2`` of the ADC clock.
Such interpolation might provide a precision better than the timestamp clock.

For the aforementioned reasons we prefer to avoid floating point numbers.
We can convert the timestamp from an integer value to a fixed-point value, that is a fractional number with a fixed number of fractional digits, see diagram :numref:`diagram-fixed-point`.
The conversion is easily obtained by multiplying the fixed-point number to a factor.
With this multiplication we decide that the last ``n`` digits are the fractional part of the timestamp.

We can determine the effect of this multiplication by taking as an example a 64 bits unsigned integer.
If we want to have a fractional precision of about 0.001 we can multiply it by 1000.
Since the digital world uses powers of two it is better if we stick to binary numbers.
The closest power of two to 1000 is 2\ :sup:`10`, which is 1024.
In binary arithmetic, a multiplication by 1024 is equivalent to a shift of 10 bits.
This way we gain 10 bits of fractional part, that are at the lowest significant end of the number, but we lose the 10 most significant bits from the number.
Effectively the number goes from a 64 bits value to a 54 bits value, limiting the maximum value of the timestamp.
The highest value that can fit in a 64 bits number is about 1.8·10\ :sup:`19`, the maximum value that can fit in a 54 bits number is about 1.8·10\ :sup:`16`.
Assuming a 1 ns step value for the ADC clock we have a maximum timestamp for the 64 bits of 1.8·10\ :sup:`10` s which is about 570 years.
For a 54 bits number the maximum is about 7 months, so for experiments that need to run longer than that the number of fractional bits should be lowered.
Using 8 bits of fractional part we get a precision of about 0.004 ns, for a 1 ns step value.
The maximum number that can fit in the other 56 bits is about 7.2\ :sup:`16`, which corresponds to 2.3 years.

Timestamps determination in ABCD
--------------------------------

The example libraries in ``waan`` are developed with these considerations in mind.
In order to increase the resolution of the timestamps, the waveforms are interpolated and the timestamps are shifted by a user-selectable number of bits.
The waveform is interpolated and its trigger position determined.
The trigger position represent the position in the waveform in which the user wants to assign its time reference.
The trigger position is summed to the timestamp provided by the digitizer.

The example libraries do not modify the original waveforms and thus waveforms can always be reanalyzed always enabling the bit shift.
If the waveform is modified by a custom library developed by a user, then the shift could already be in the timestamp value and thus the shift should not be repeated.
This is why most timestamp determination libraries have the possibility of disabling the bit shift.
Some digitizer interfaces also apply a bit shift to the recorded timestamps, because some digitizers might already provide an interpolated value for the timestamp (*e.g.* the Constant Fraction Discrimination algorithm in the CAEN DT5730).
For these cases the bit shift should be disabled as well.

Case study: timestamps from ADQ36 digitizers with different sampling frequencies
--------------------------------------------------------------------------------

.. code-block:: none
    :name: diagram-timestamp-determination-ADQ36
    :caption: Diagram of the determination of the timestamp in an ADQ36 digitizer.

                         ...  <= Waveform signal
                        .   ..
         ADC samples => X     .
                       .       ..
                       .         ..  
                      .            X..  
                      .               ...
                     .                   ..
                    .                      ..X
     Threshold => - . - - - - - - - - - - - - -.- - - - - - - - - - - - - - - -
         level     .|                           ....
                 ..                                 ...X...
     X.........X.   |                                      ......X.........X...
     +----+----+----+----+----+----+----+----+----+----+----+----+----+----+---
     |    |    |    |    |    |    |    |    |    |    |    |    |    |    |   
     N   N+1  N+2  N+3 <= Timestamp clock ticks  ...  ...  ...  ...  ...  ...  
                    ^
                    Threshold crossing clock tick = T
     |--------------| <= Record start value (negative) = Δٖt_start
     +---------+---------+---------+---------+---------+---------+---------+---
     |         |         |         |         |         |         |         |
     M        M+1       M+2 <= ADC samples ticks      ...       ...       ...
                    ^
                    Interpolated threshold crossing sample = t_0

Let us consider the practical example of the ADQ36 digitizers from SP Devices.
These digitizers have two possible functioning modes:

* **4 channel mode**: All four channels sample the signals with a frequency :math:`\nu_{4\text{ch}} = 2.5\ \text{GHz}` which corresponds to a temporal step :math:`\Delta t_{4\text{ch}} = 400\ \text{ps}`;
* **2 channel mode**: Only two of the channels are active and they sample the signals with a frequency :math:`\nu_{2\text{ch}} = 5\ \text{GHz}` which corresponds to a temporal step :math:`\Delta t_{2\text{ch}} = 200\ \text{ps}`.

The digitizer also provides a timestamp :math:`T` of the threshold crossing point, see diagram :numref:`diagram-timestamp-determination-ADQ36`, with a temporal step :math:`\delta t = 25\ \text{ps}`.
The relationships between the temporal steps are:

.. math::

   \Delta t_{4\text{ch}} = 400\ \text{ps} &=  16\cdot \delta t = (\delta t \ll 4\ \text{bit}) \\
   \Delta t_{2\text{ch}} = 200\ \text{ps} &=  8\cdot \delta t = (\delta t \ll 3\ \text{bit})

Where the symbol ":math:`\ll`" represents a bit shift of the binary numbers.

Finally the digitizer provides also a *record start* value :math:`\Delta t_{\text{start}}`, which represents the time span between the threshold crossing point and the actual start of the sampled waveform, see diagram :numref:`diagram-timestamp-determination-ADQ36`.
The record start has the same temporal step of the timestamps, so :math:`\delta t = 25\ \text{ps}`.
In the case of :numref:`diagram-timestamp-determination-ADQ36`, the record start is a negative number (:math:`\Delta t_{\text{start}} < 0`), because the waveform start is before the threshold crossing point.

Let us take as the reference the 4 channel mode.
We want to impose an **8 bits** **fractional part** to the timestamps.
Then the reference temporal step :math:`\tau`, that we are selecting, is:

.. math::

   \tau = (\Delta t_{4\text{ch}} \gg 8\ \text{bit}) = \frac{\Delta t_{4\text{ch}}}{256} = \frac{400\ \text{ps}}{256} = 1.5625\ \text{ps}

We then need to rescale the temporal steps to this reference value:

* **timestamp**: :math:`\delta t = 25\ \text{ps} = 16 \tau = (\tau \ll 4\ \text{bit})`.
* **record start**: :math:`\delta t = 25\ \text{ps} = 16 \tau = (\tau \ll 4\ \text{bit})`.
* **4 channel mode sampling**: :math:`\Delta t_{4\text{ch}} = 400\ \text{ps} = 16\cdot \delta t = 256 \tau = (\tau \ll 8\ \text{bit})`.
* **2 channel mode sampling**: :math:`\Delta t_{2\text{ch}} = 200\ \text{ps} = 8\cdot \delta t = 128 \tau = (\tau \ll 7\ \text{bit})`.

In order to achieve a better temporal resolution it is always possible to interpolate the waveform's samples.
In the case of :numref:`diagram-timestamp-determination-ADQ36`, the threshold-crossing sample :math:`t_0` is between samples ``M+1`` and ``M+2`` of the ADC sampling.
Since :math:`t_0` is interpolated, its value would be fractional with a temporal scale of :math:`\Delta t_{4\text{ch}}` or :math:`\Delta t_{2\text{ch}}` (depending on the functioning mode).

In ABCD the timestamps are determined in order to have a uniform representation for all the channels as a 64 bits unsigned integer, for instance for finding temporal coincidences.
If there are multiple digitizers with the different modes of operation then the timestamps need to be rescaled:

.. math::

   T_{\text{ABCD}} &= \big[(T + \Delta t_{\text{start}}) \ll 4\ \text{bit}\big] + (t_0 \ll 8\ \text{bit}) \quad \text{(4 channel mode)} \\
   T_{\text{ABCD}} &= \big[(T + \Delta t_{\text{start}}) \ll 4\ \text{bit}\big] + (t_0 \ll 7\ \text{bit}) \quad \text{(2 channel mode)}

This calculation can be fully done at the ``waan`` analysis level or separated in two steps:

1. At the module reading the data from the digitizer (``absp``) we shift the timestamps to the decided temporal scale, so all the timestamps would then be coherent.
   Practically speaking, in the ``absp`` then we should set a bit shift of 4 bits.
   A word of warning, the sum between :math:`T` and :math:`\Delta t_{\text{start}}` can only be done at the ``absp`` level, because the :math:`\Delta t_{\text{start}}` information is discarded before the data is sent.
2. At the ``waan`` analysis level we use the specific shift for the interpolated values of :math:`t_0`, so 7 or 8 bits depending on the sampling of the channel.
   At the ``waan`` stage it is important to disable the bit shift to the waveform original timestamp :math:`T`, because it was already done at the ``absp`` level.
   The analysis function will only multiply the :math:`t_0` value by the chosen bit shift.

This example follows the usual approach in the example libraries of ``waan``, but the user is always welcome to customize and fit them to the specific needs.