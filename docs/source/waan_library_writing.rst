.. _ch-waan-library-writing:

=================================================
Advanced topic: writing a simple ``waan`` library
=================================================

In this tutorial we write a simple library for ``waan`` that determines the energy of a waveform by a simple intregration.
The average of the first *N* samples will be the **baseline** of the waveform.
Two integration gates will be used, in order to apply the *double integration method* for Pulse Shape Discrimination (PSD).
This tutorial will simply guide the reader through the writing of this simple library, but it will not go into the implementation details of ``waan``.
See :numref:`ch-waan` for more information.
See :numref:`sec-waveforms-analysis-page` for a detailed description of the ``waan`` user interface.

.. code-block:: none
    :name: diagram-double-integration
    :caption: Simplified diagram of a waveform with the integration domains of the *double integration method* for Pulse Shape Discrimination (PSD).

                  oo    oo  oo                                               oo
    Baseline -> oo-o--oo-o-o-|o------|-------------------------------|----ooo--o
       level        oo    oo   o                             oo  ooooooooo
                |------------| o     |                oo  ooo  oo    |
                  Baseline:    o                   ooo  oo
                  Average of | o     |        ooooo                  |
                   first N      o          ooo
                   samples   |  o    |  ooo                          |
                                o      o
                             |  o    oo                              |
                                o    o
                             |  o   o|                               |
                                 o  o 
                             |   o o |                               |
                                  o                                   
                qshort ->    |-------|                               |
                qlong  ->    |---------------------------------------|
                                ^^^ Integrations domains ^^^

Introduction to the library
---------------------------

``waan`` expect to find a shared library file containing the function that it will apply to every waveform.
The library shall have a C99 interface.
In principle it is possible to write the library in C++ and exposing a C interface, but we will not follow that route in this tutorial.
Do it at your own enjoyment.
Thus we are going to write a ``.c`` file which contains the function that will analyze the waveforms.
``waan`` will apply this function to every waveform, thus it should be as lightweight as possible.

This function can retain information between subsequent calls in the memory pointed by ``user_config``, but the management of that information is the user's responsibility.
The function cannot access all the waveforms at the same time, only the one that it received.
Since this function is executed very frequently we suggest not to access files in the function.
For instance reading a configuration from a file in the ``energy_analysis()``, would be repeated for every waveform and this is very wasteful of resources.
It is better to use the ``energy_init()`` function for reading the parameters.
In this simple example we will `hard-code <https://en.wikipedia.org/wiki/Hard_coding>`_ the configuration parameters in the source code of the library as constants.
This is not a good practice, in general, because we would have to recompile the library for every change of the parameters values.
`Soft-coding <https://en.wikipedia.org/wiki/Hard_coding>`_ is a much better idea.
In fact there is the possibility of passing configuration parameters to the user libraries, directly from the ``waan`` configuration files.
For this we suggest to look at the `example libraries <https://en.wikipedia.org/wiki/Hard_coding>`_.
The configuration is stored in a JSON object of the `jansson <https://github.com/akheron/jansson>`_ library.
Since this would require a much longer tutorial, we opted to the quicker solution of hard-coding the parameters.

.. warning::

    **With great power comes great responsibility.**

    The ``waan`` module will load and execute whatever the user asks.
    Since these are function written in C99, there is always the chance that some `memory leak <https://en.wikipedia.org/wiki/Memory_leak>`_ or `segfault <https://en.wikipedia.org/wiki/Segmentation_fault>`_ errors can go unnoticed.
    If the error is really disastrous there is the good chance that ``waan`` would crash as soon as it analyzes the first waveform.
    The symptom of a memory error is the disappearance of the ``waan`` process.
    In the user interface the ``waan`` interface would show a connection error with ``waan`` (after some delay).
    There is also the possibility of a subtle error that happens once in a while, or whenever a waveform is really pathologic.
    These are of course the worst errors, because the ``waan`` process would disappear randomly with no evident error.
    Be very careful when you develop a library for ``waan`` and check all the possible memory errors.

Preamble of the library
-----------------------

For this simple example we need to include only a few headers.

.. code-block:: c

    #include <stdint.h>
    #include <stdbool.h>

    #include "analysis_functions.h"
    #include "events.h"

The standard header ``stdint.h`` is used to define the portable data types with a fixed width (such as ``uint16_t``).
The whole framework is implemented using those data types to guarantee the intercomunication between modules and different programming languages.
The standard header ``stdbool.h`` is used to define the constants ``true`` and ``false`` and the ``bool`` data type.
The header ``analysis_functions.h`` contains the definitions of the functions that ``waan`` uses.
That header file is commented in details and we suggest to read it to understand better the implementation of ``waan`` libraries.
Finally ``events.h`` contains the definition of the data types used by ABCD (*i.e.* the PSD events and the waveforms).

Parameters
----------

Analysis routines require some configuration parameters in their functioning (*e.g.* integration width).
As mentioned earlier, we will hard-code the parameters in the source code.

.. code-block:: c

    /*! \brief The number of samples to be averaged at the pulse begin, in order to
     *         determine the baseline.
     */
    #define BASELINE_SAMPLES 64

    /*! \brief Defining if the pulse is positive.
     */
    #define POSITIVE_PULSE false

    /*! \brief The start position of the integration gates.
     */
    #define INTEGRATION_START 110

    /*! \brief The short integration gate.
     */
    #define GATE_SHORT 30

    /*! \brief The long integration gate.
     */
    #define GATE_LONG 90

Analysis function
-----------------

Here we get to the kernel of the library, the function that is actually applied to the waveforms.

.. code-block:: c

    /*! \brief Function that determines the energy information.
     */
    void energy_analysis(const uint16_t *samples,
                         uint32_t samples_number,
                         struct event_waveform *waveform,
                         uint32_t **trigger_positions,
                         struct event_PSD **events_buffer,
                         size_t *events_number,
                         void *user_config)
    {
        UNUSED(user_config);
    
        // Assuring that there is one event_PSD and discarding others
        reallocate_buffers(trigger_positions, events_buffer, events_number, 1);

Here we see the declaration of the function.
It receives a pointer to the samples of the waveform, that is an array of ``uint16_t`` values.
The length of the array is defined by ``samples_number``.
The function receives the actual ``event_waveform`` that contains the ``samples``.
This is a redundant information that allows the user to modify the waveform in case it is desired.
The ``trigger_poisitons`` is an array of ``uint32_t`` values.
It contains all the positions of physical events that were detected in the waveform.
The ``trigger_positions`` determination is carried out by the ``timestamp_analysis()`` function, that can also be customized by the user.
The ``events_buffer`` is an output array with the same number of elements as the ``trigger_positions`` array.
The number of elements in the ``trigger_positions`` and ``events_buffer`` is defined in ``events_number``.
The reasoning is that for every trigger position there should be one physical event in the waveform, and thus an output event for it.
The ``trigger_positions`` and ``events_buffer`` arrays are managed with the ``reallocate_buffers()`` function.
In this example we call the ``reallocate_buffers()`` function in order to make sure that there is one and only one output event.

Analysis algorithm
------------------

First we determine the baseline by averaging the first *N* samples.

.. code-block:: c

    // Calculating the baseline
    double baseline = 0;

    for (uint32_t i = 0; (i < BASELINE_SAMPLES) && (i < samples_number); i++) {

        baseline += samples[i];
    }

    baseline /= BASELINE_SAMPLES;

To make sure that we do not read the memory outside the samples array we are using the condition in the loop: ``(i < BASELINE_SAMPLES) && (i < samples_number)``
If the waveform is too short (shorter than the *N* samples to average) the result would be obviously not very reliable, but at least we would not risk of reading memory locations that we are not supposed to.

We then calculate the integrals.

.. code-block:: c

    // Calculating the integrals
    double qshort = 0;

    for (uint32_t i = INTEGRATION_START; (i < (INTEGRATION_START + GATE_SHORT)) && (i < samples_number); i++) {

        if (POSITIVE_PULSE) {
            qshort += (samples[i] - baseline);
        } else {
            qshort += (baseline - samples[i]);
        }
    }

    double qlong = 0;

    for (uint32_t i = INTEGRATION_START; (i < (INTEGRATION_START + GATE_LONG)) && (i < samples_number); i++) {

        if (POSITIVE_PULSE) {
            qlong += (samples[i] - baseline);
        } else {
            qlong += (baseline - samples[i]);
        }
    }

Output
------

To output the result of this calculation we need to write the two integrals to the output events in the ``events_buffer`` array.
Since we used the function ``reallocate_buffers()`` we can be sure that there is at least one ``event_PSD`` in the output array to which we can write the information.

.. code-block:: c

    // Output of the function
    (*events_buffer)[0].timestamp = waveform->timestamp;
    (*events_buffer)[0].qshort = qshort;
    (*events_buffer)[0].qlong = qlong;
    (*events_buffer)[0].baseline = baseline;
    (*events_buffer)[0].channel = waveform->channel;
    (*events_buffer)[0].group_counter = 0;

In this example we are overwriting the ``timestamp`` entry, using the original one from the waveform.
If the waveform was analyzed before by the ``timestamp_analysis()`` function, then the ``timestamp`` entry of the ``event_PSD`` might be already there.
If you are using the ``timestamp_analysis()`` library, then you should not overwrite the ``timestamp`` entry.

Compilation
-----------

We can compile our custom library with the `Makefile <https://github.com/ec-jrc/abcd/blob/main/waan/Makefile>`_ that is in the ``waan`` folder.
We need to pass to the Makefile the full path of the file, making sure that we change the extension from ``.c`` to ``.so``::

    user-tutorial@abcd-tutorial:~/abcd/waan$ make src/libSimplePSD.so

Where the source code of the library would be ``src/libSimplePSD.c``.
The resulting function of this tutorial is in the ABCD repository: https://github.com/ec-jrc/abcd/blob/main/waan/src/libSimplePSD.c