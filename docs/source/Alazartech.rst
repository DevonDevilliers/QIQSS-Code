Alazartech Acquisition card
===========================

Introduction
------------
Card developed by Alazartech for the acquisition of signals: `Link text <https://www.alazartech.com/en/product/ats9462/13/>`_.
It is used to acquire signals, and to analyse them to perform FFT, PSD or a Rabi experiment.

Installation 
------------
The code for the control of Alazartech acquisition cards is part of PyHegel, to be installed following `Link text <https://github.com/lupien/pyHegel/blob/master/README.rst>`_.
The code is in the 'instruments' package of PyHegel, under the name 'Alazartech.py'. 
You can detect all the Alazartech acquisition cards connected with the computer using the 'find_all_Alazartech' function.

>>> instruments.find_all_Alazartech()

If cards are connected together, they form a system. The function returns a dictionary having for keys the labels of the system and for values a tuple with the number of cards in the system and a new dictionary with keys the labels of the card in the system and values the main information about the card.
All cards are represented by the id number of their system and their id in this system. You can connect with any cards detected using 'instruments.find_all_Alazartech()' by using the function 'instruments.ATSBoard(systemId, boardId)'

>>> ats = instruments.ATSBoard()

In our case, we have only one card Alazartech, so its systemId and boardId are both equal to 1. If no cards are detected using 'find_all_Alazartech', the installation of the card should be checked.
First, check if the card is correctly wired on the computer, then check if its pilots are well installed and finally if the SDK version should not be updated. Information about the installation of the card is found on the Alazartech website, and information about the version of the SDK and Driver are obtained using 'instruments.find_all_Alazartech'. 

Acquiring signals
-----------------
On the back of the Alazartech card, 5 ports can be connected with coaxial cables. 
- The ports "CHA" & "CHB" acquire signals on Channel A and Channel B
- the port "TRIG IN" is to receive a signal used to trigger acquisition of channel A and/or B
- the "AUX I/O" port is to synchronize instruments by either supplying a 5V TTL-level signal or receiving a TTL-level input signal   
- the "ECLK" port is to acquire signals on channel A and/or B with a sample rate between 150 and 180 MHz in 1MHz step by receiving  