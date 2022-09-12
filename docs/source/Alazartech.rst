Alazartech Acquisition card
***************************

Introduction
============

Card developed by Alazartech for the acquisition of signals: `Alazartech website <https://www.alazartech.com/en/product/ats9462/13/>`_.
It is used to acquire signals, and to analyse them to perform FFT, PSD or a Rabi experiment.

Installation 
============

The code for the control of Alazartech acquisition cards is part of PyHegel, to be installed following the instructions on `PyHegel repository <https://github.com/lupien/pyHegel/blob/master/README.rst>`_.
The code is in the :mod:`instruments` module of PyHegel, under the name 'Alazartech.py'. 
You can detect all the Alazartech acquisition cards connected with the computer using the :py:func:`find_all_Alazartech` function.

>>> instruments.find_all_Alazartech()

If cards are connected together, they form a system. The function returns a dictionary having for keys the labels of the system and for values a tuple with the number of cards in the system and a new dictionary with keys the labels of the card in the system and values the main information about the card.
All cards are represented by the id number of their system and their id in this system. You can connect with any cards detected using :py:func:`instruments.find_all_Alazartech()` by using the function :py:func:`instruments.ATSBoard(systemId=1, boardId=1)`

>>> ats = instruments.ATSBoard()

In our case, we have only one card Alazartech, so its systemId and boardId are both equal to 1. If no cards are detected using :py:func:`instruments.find_all_Alazartech`, the installation of the card should be checked. First, check if the card is correctly wired on the computer, then check if its pilots are well installed and finally if the SDK version should not be updated. Information about the installation of the card is found on the Alazartech website, and information about the version of the SDK and Driver are obtained using :py:func:`instruments.find_all_Alazartech`. 

.. _Alazartech Overview:
Overview of the functionalities
===============================

Working with PyHegel
--------------------

From the iPython interface of PyHegel, you can create an Alazartech card ``instrument`` using :py:func:`instruments.ATSBoard`. This instrument is an instance of the Python class :class:`instruments.ATSBoard`. In this instance, some ``devices`` are defined, that are interactive attributes of the instrument. In the following, all the ``devices`` of :py:func:`instruments.ATSBoard` are written as :class:`device`. The minimum function of ``devices`` is to store data. The value of a ``device`` :class:`device` is obtained via its method :py:func:`get`. The :py:func:`get` method is defined in pyHegel and has many features, including the possibility to write the data in a file by defining a filename in the ``filename`` option. The file is saved in the ".txt" format, but it can be saved in another format by defining it in the ``bin`` option. See PyHegel documentation, run ``get?`` or ``get??`` in the PyHegel console to see the get options. Generally speaking, you can access the description of any ``device`` by running ``device?`` and see the source code of the device by running ``device??``.

>>> ats = instruments.ATSBoard()  # ats is an instrument 
>>> ats.acquisition_length_sec.get()  # print the value of the device acquisition_length_sec
>>> get ats.acquisition_length_sec, filename="acquisition_length_sec.txt"  # print the value of the device acquisition_length_sec and store it in a .txt file
>>> ats.acquisition_length_sec?  # see documentation associated with device acquisition_length_sec

All the devices and their value can be obtained by running :py:func:`ipython`. The value of certain devices can be changed interactively using the :py:func:``set`` method.

>>> set ats.acquisition_length_sec, 1e-3  # device acquisition_length_sec now stores the value 1e-3
>>> ats.acquisition_length_sec.set(1e-3)  # same operation

.. _Alazartech Overview Function:
Devices and functionalities of the card
---------------------------------------

The main functionality of the card is to acquire signals. Acquisition are of different types, see :ref:`the acquisition types<Alazartech acquisition types>`), and are defined by their duration :class:`acquisition_length_sec`, the number of points per acquisition :class:`samples_per_record` and the sample rate :class:`sample_rate`. You can acquire up to two streams of data and trigger using these channels or an external signal (see :ref:`acquisition ports<Alazartech acquisition types>`). The acquisitions are performed as on an oscilloscope and you need to define the size of the screen with :class:`input_range`, and the mode of the screen with :class:`acquisition_mode`. You can also modify some :ref:`inner parameters of the acquisition<Alazartech acquisition inner>`.

The card can also be used to post-process signals. The FFT or the PSD of any acquired signal can be made using :py:func:`instruments.ATSBoard.make_fft` or :py:func:`instruments.ATSBoard.make_psd`. The FFT or the PSD of a channel defined in :class:`current_channel` can be directly performed using :class:`fft` or :class:`psd`. The main parameters for these post-processing are the number of points taken to perform FFT :class:`psd_fft_lines`, the sample rate, the unit of the output :class:`psd_units` and the starting and ending frequencies at which the FFT/PSD should be shown, :class:`psd_start_freq` and :class:`psd_end_freq`.

It can also be used to analyse the results of a Rabi experiment using :class:`rabi`. After acquiring :class:`nbwindows` acquisitions, :py:func:`instruments.ATSBoard.detection_threshold` detects the times at which the signal crosses a threshold in a descending or ascending way (:class:`trigger_level_descend` and :class:`trigger_level_ascend`). It is possible to smooth curves using a sliding mean method involving :class:`nbpoints_sliding_mean` points with :py:func:`instruments.ATSBoard.smooth_curve`. 

.. _Alazartech Acquisition:
Acquiring signals
=================

.. _Alazartech Acquisition ports:
Acquisition ports
-----------------

On the back of the Alazartech card, 5 ports can be connected with coaxial cables. 

* The ports ``CHA`` & ``CHB`` acquire signals on Channel A and Channel B.
* the port ``TRIG IN`` is to receive a signal used to trigger acquisition of channel A and/or B.
* the ``AUX I/O`` port is to synchronize instruments by either supplying a 5V TTL-level signal or receiving a TTL-level input signal.   
* the ``ECLK`` port is to acquire signals on channel A and/or B with a sample rate between 150 and 180 MHz in 1MHz step by receiving.  

It is possible to acquire either channel A or channel B or both of them. The device :class:`active_channels` defines a list of the channels to acquire. For each channel to acquire, :class:`trigger_channel_1` and :class:`trigger_channel_2` defines which port to use as a reference for the trigger operation, and :class:`trigger_level_1` and :class:`trigger_level_2` stores the level at which to trigger in mV. The acquisition can start when the signal reaches the trigger level in an ascending or descending way, which is defined in :class:`trigger_slope_1` and :class:`trigger_slope_2`.

The internal clock of Alazartech cards enables the acquisitions to be performed only for certain sample rates. These sample rates depend on the acquisition card, but they range from 1KS/s to 4GS/s for the best cards. You can choose between using an internal clock ``INT`` and an external clock ``EXT`` by defining :class:`clock_type`.  

The ``AUX I/O`` port can be configured using :class:`aux_io_mode` and :class:`aux_io_param`.

.. _Alazartech acquisition types:
Acquisition types
-----------------

Three types of acquisition can be performed.

* :py:func:`readval.get` performs a trigger operation: it acquires the signal for a duration :class:`acquisition_length_sec` at a sampling rate :class:`sampling_rate` when the signal reaches a threshold current. :py:func:`fetch.get` works similarly. If an acquisition has already been performed for the channels defined in :class:`active_channels`, it simply outputs the result of this acquisition. Otherwise, :py:func:`fetch.get` calls :py:func:`readval.get`.
* :py:func:`readval_all.get` performs several trigger operation faster than by repeating :py:func:`readval.get`. The number of acquisitions is defined by device :class:`nbwindows` and the channels to acquire are defined by :class:`active_channels`. The function :py:func:`fetch_all.get` works similarly, except it only gets data for one channel, the :class:`current_channel`.
* :py:func:`continuous_read.get` acquires a signal right after being called. It calls :py:func:`readval.get` but with a :class:`trigger_mode` set to ``Continuous`` instead of ``Triggered``.

The sampling rate, the acquisition time and the number of points per records are related as :math:`sample_rate * acquisition_length_sec = samples_per_record`. :class:`sample_rate` can only take some values, and a finer control can be obtained by using the ``ECLK`` port. Changing the sample rate impacts the number of samples per record without changing the duration of the acquisition. Changing the duration of the acquisition also impacts the number of samples per record without changing the sample rate.

.. _Alazartech acquisition inner:
Structure of an acquisition
---------------------------

Before actually performing the acquisition, some operations have to be realized. The card should first be configured, which means that the :class:`sample rate`, the :class:`clock_type` used for the sampling, the size of the screen :class:`input_range`, the delay to trigger :class:`trigger_delay` should be provided to the card, alongside information about each channel 1 (channel A) and 2 (channel B), that are the channel used to trigger :class:`trigger_channel_1`/:class:`trigger_channel_2` , the level at which to trigger :class:`trigger_level_1`/:class:`trigger_level_2`, the slope for which to trigger (for an ascending or descending signal) :class:`trigger_slope_1` and :class:`trigger_slope_2`, and the limit of the bandwith :class:`bw_limited_A`/:class:`bw_limited_B`. This configuration is performed in :py:func:`instruments.ATSBoard.ConfigureBoard`. When calling this function, the board is configured if the parameters have never been provided to the card or if one parameter was modified since the previous configuration. This function is called at the beginning of all the acquisition functions.    

