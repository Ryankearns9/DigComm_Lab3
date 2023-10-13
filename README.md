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

The repeat block will replicate an input N times. This creates an oversampling ratio of N such that there are N samples per symbol. In this case, the oversampling ratio was chosen to be 600. This ratio was chosen to allow for a clean looking time-domain plot and also allows for easier representation without overmodulation.

#### Frequency Modulation
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/FrequencyMod.PNG)

The frequency modulation translates changes in input amplitude to changes in frequency. The equation which describes this translation is shown below $e^{j2\pi*f_\delta/f_s * {\sum{x[n]}}}$ where $f_\delta$ is the frequency deviation, $f_s$ is the sample rate and x[n] is the amplitude of the Nth sample as per the documentation (https://wiki.gnuradio.org/index.php/Frequency_Mod).

This module accepts a single argument, the frequency sensitivity which is defined by $2\pi*f_\delta/f_s$. This value is fed by the GUI Range Variable "freq_deviation" and ranges between 1kHz and 200kHz. As shown in the modulation plots below, this value is dictating how much frequency will vary when amplitude changes. A deviation of 200kHz means that a -1 will be represented by a tone 200kHz below the carrier while a +1 will be represented by a tone 200kHz above the carrier. Larger deviation results in less bit error but also comes results in a higher required bandwidth. An engineer must trade how much bandwidth is required with the the amount of allowable bit error.

The input of this module is a floating point value while the output is a complex value.

#### Mixer
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Mixer.PNG)

The mixer module consists of the modulated signal being multipled by a cosine value. The resulting output frequency is $f_{cosine}+f_{mod}$. This gives the cosine value the property of being the carrier frequency. As the previous section details the modulation frequecy output being +/- $f_\delta$, we know that the final frequency will be given as $f_{cosine}+X[N]*f_\delta$.

##### Signal Generator
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/MixingSource.PNG)

The cosine tone generated for the mixer is from the signal generator pictured above. It produces a cosine tone of a given frequency. In this case, it generates a 315MHz sampled at 8MHz. Given that the carrier frequency is higher than our sample rate, this results in the signal aliasing to 3MHz due to the nature of complex sampling.

##### Throttle
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Throttle.PNG)

The throttle controls how fast a computer can copy memory from one location to another. Given that there is no hardware input controling timing, this module ensures the computer reduces its processing speed to the rate defined by the throttle. In this case, it is 8MHz.

### Modulation GUI
The below two sections detail the modulation output with 10kHz deviation and 200kHz deviation. This helps visualize how frequency deviation affects the output spectrum.

#### High Deviation
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/ModulatedOutput_HighDeviation.PNG)

The above figure shows the modulation with 200kHz deviation. As shown on the waterfall, there are two distinct tones present at 3.2MHz and 2.8MHz. These are the expected locations gives in the Mixer section.

#### Low Deviation
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/ModulatedOutput_LowDeviation.PNG)

The above figure shows the modulation with 10kHz deviation. As shown on the waterfall and FFT, these tones are nearly impossible to distinguish due to their close proximity in frequency. The bit error rate from this modulation would likely be significantly higher than the previous example.


