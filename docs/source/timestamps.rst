.. _ch-timestamps:

========================
Timestamps determination
========================

Introduction
------------

Most digitizers provide a timestamp, that is the characteristic time information associated with a waveform.
Normally it is a value obtained with a digital counter, that provide integer numbers.
This counter counts the number of cycles of the clock of the ADC.
Being an integer it is an exact number and thus its precision is the very same as the ADC's clock.
Normally the clocks runs at frequencies of the order of few hundreds of MHz or a few GHz.
Assuming that the clock is 1 GHz, for simplicity, the time step would be of the order of 1 ns.
Thus after 1 s of acquisition time the value of a timestamp can be of the order of 1 billion (10 :sup:`9`).
Since in some experiments we want to measure time differences of the order of 10 ns or less, we need a very high precision on the timestamp number.
Floating point values are approximated values and thus they might lose the needed precision.
For these reasons, most of the times timestamps are stored in 64 bits unsigned integers.

Fixed-point values
------------------

The digitizers normally provide the timestamp of the threshold crossing sample, that has the precision of a clock step.
There is the possibility of obtaining a better precision by interpolating the samples.
The best precision that an integer number can provide is one clock sample, thus we need to improve that precision (especially for the slower digitizers).
We can convert the timestamp from an integer value to a fixed-point value, that is a fractional number with a fixed number of fractional digits.
The conversion is easily obtained by multiplying the fixed-point number to a factor.
With this multiplication we decide that the last N digits are the fractional part of the timestamp.

We can determine the effect of this multiplication by taking as an example a 64 bits unsigned integer.
If we want to have a fractional precision of about 0.001 we can multiply it by 1000.
Since the digital world uses powers of two it is better if we stick to those.
The closest power of two to 1000 is 2 :sup:`10`, which is 1024.
In binary arithmetic, a multiplication by 1024 is equivalent to a shift of 10 bits.
This way we gain 10 bits of fractional part, that are at the lowest significant end of the number, but we lose the 10 most significant bits from the number.
Effectively the number goes from a 64 bits value to a 54 bits value, limiting the maximum value of the timestamp.
The highest value that can fit in a 64 bits number is about 1.8·10 :sup:`19`, the maximum value that can fit in a 54 bits number is about 1.8·10 :sup:`16`.
Assuming a 1 ns step value for the ADC clock we have a maximum timestamp for the 64 bits of 1.8·10 :sup:`10` s which is about 570 years.
For a 54 bits number the maximum is about 0.6 years, so for experiments that need to run longer than that a single timestamp would not be sufficient.

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