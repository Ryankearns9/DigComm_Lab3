# Lab 3

## Introduction
Lab 3's objective is to build an end-to-end FSK Simulation. It includes both the modulation and demodulation portions of the FSK path. FSK is the encoding of binary information into the frequency deomain information. Most commonly, specific frequency values will translate to different distinct frequencies.

## Equipment List
The following section outlines the equipment used in this lab. All equipment can be purchased following the links provided

### Hardware
1. ASUS Laptop
 - https://www.amazon.com/Performance-Premium-Processor-SuperMulti-10-Silver/dp/B01KAP7RJG

### Software
1. GNU Radio
 - https://www.gnuradio.org/
 
### Referenced Labs
Lab 1
https://github.com/Ryankearns9/DigComm_Lab1

Lab 2
https://github.com/Ryankearns9/DigComm_Lab2

## Algorithm Description
This section details the specific algorithm implimented in GNU Radio. It includes the overall flow diagram along with specific descriptions of each part.

### Flow Diagram
The below figure shows the overall flow diagram of the algorithm. It includes the layout for both the modulation, demodulation, and constants used for the simulation. The below sections will detail these in more detail.
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Flow_diagram.PNG)

### Modulation
The below section shows the modulation from vector source through to frequency mixing. The key sections of this simulation is the Vector Source, repeater, frequency modulator, and mixer. 

![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Rx_Flow.PNG)

#### Vector Source
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/VectorSource.PNG)

The vector source is the symbol list used for the FSK signal. This is a randomly generated list of symbols alternating between -1 and +1 where -1 represents a binary 0 and +1 represents a binary 1. This list is repeated to allow for the simulation of an arbritrarily long data vector.

#### Repeat Block
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Repeat.PNG)

The repeat block will replicate an input N times. This creates an oversampling ratio of N such that there are N samples per symbol. In this case, the oversampling ratio was chosen to be 600. This ratio was chosen to allow for a clean looking time-domain plot.

#### Frequency Modulation
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/FrequencyMod.PNG)

The frequency modulation translates changes in input amplitude to changes in frequency. The equation which describes this translation is shown below $e^{2*\pi*f_\delta/f_s}$