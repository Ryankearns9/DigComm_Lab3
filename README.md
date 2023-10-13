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

## Flow Diagram
The below figure shows the overall flow diagram of the algorithm. It includes the layout for both the modulation, demodulation, and constants used for the simulation. The below sections will detail these in more detail.
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Flow_diagram.PNG)

## Modulation
The below section shows the modulation from vector source through to frequency mixing. The key sections of this simulation is the Vector Source, repeater, frequency modulator, and mixer. 

![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Tx_Flow.PNG)

### Vector Source
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/VectorSource.PNG)

The vector source is the symbol list used for the FSK signal. This is a randomly generated list of symbols alternating between -1 and +1 where -1 represents a binary 0 and +1 represents a binary 1. This list is repeated to allow for the simulation of an arbritrarily long data vector.

### Repeat Block
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Repeat.PNG)

The repeat block will replicate an input N times. This creates an oversampling ratio of N such that there are N samples per symbol. In this case, the oversampling ratio was chosen to be 600. This ratio was chosen to allow for a clean looking time-domain plot and also allows for easier representation without overmodulation.

### Frequency Modulation
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/FrequencyMod.PNG)

The frequency modulation translates changes in input amplitude to changes in frequency. The equation which describes this translation is shown below $e^{j2\pi*f_\delta/f_s * {\sum{x[n]}}}$ where $f_\delta$ is the frequency deviation, $f_s$ is the sample rate and x[n] is the amplitude of the Nth sample as per the documentation (https://wiki.gnuradio.org/index.php/Frequency_Mod).

This module accepts a single argument, the frequency sensitivity which is defined by $2\pi*f_\delta/f_s$. This value is fed by the GUI Range Variable "freq_deviation" and ranges between 1kHz and 200kHz. As shown in the modulation plots below, this value is dictating how much frequency will vary when amplitude changes. A deviation of 200kHz means that a -1 will be represented by a tone 200kHz below the carrier while a +1 will be represented by a tone 200kHz above the carrier. Larger deviation results in less bit error but also comes results in a higher required bandwidth. An engineer must trade how much bandwidth is required with the the amount of allowable bit error.

The input of this module is a floating point value while the output is a complex value.

### Mixer
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Mixer.PNG)

The mixer module consists of the modulated signal being multipled by a cosine value. The resulting output frequency is $f_{cosine}+f_{mod}$. This gives the cosine value the property of being the carrier frequency. As the previous section details the modulation frequecy output being +/- $f_\delta$, we know that the final frequency will be given as $f_{cosine}+X[N]*f_\delta$.

#### Signal Generator
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/MixingSource.PNG)

The cosine tone generated for the mixer is from the signal generator pictured above. It produces a cosine tone of a given frequency. In this case, it generates a 315MHz sampled at 8MHz. Given that the carrier frequency is higher than our sample rate, this results in the signal aliasing to 3MHz due to the nature of complex sampling.

#### Throttle
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

## Demodulation
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Rx_Flow.PNG)

The above figure shows the demodulation flow of this FSK module. It consists of a FIR filter, quadrature demodulator, constant add, binary slicer, and a UChar to float convertor.

This module accepts the FSK output from the above section and converts it to a binary output where the -1/+1 symbols are translated to 0/1 binary data. 

### Frequency Xlating FIR Filter
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/FIR_Filter.PNG)

This type of FIR filter is a mixer followed by a FIR filter then a downsampler. The mixer basebands a signal of interest prior to filtering. The FIR filter is then applied in the same way as described in Lab 1. Finally, the decimator works the same as described in Lab 1. The module accepts a cener frequency, FIR taps, and decimation amount.

The mixer frequency is chosen based on the carrier center frequency. In this case, it is 315MHz which will be aliased to 3MHz.

The FIR taps are generated by the function firdes.low_pass as decribed in its documentation here: https://www.gnuradio.org/doc/doxygen/classgr_1_1filter_1_1firdes.html. The filter accepts the following parameters:

| Parameter        | Value         |
| ---------------- |:-------------:|
| Fitler Type      | Hamming       |
| Gain             | 1.0           |
| Sampling Rate    | 8MHz          |
| Cutoff Freq      | 7.5kHz        |
| Transition Width | 1kHz          |

Of note, the passband edge for this filter is 7.5kHz which means any frequency deviation above this value will be significantly attenuated any may not be recoverable. This can be seen with the below two figures

![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Waterfall_inband.PNG)

The above figure shows expected output for a frequency deviation of 6kHz with high energy density within the 7.5kHz passband of the filter.

![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Waterfall_outofband.PNG)

The above figure shows the extremely attenuated output from frequency deviation of 156kHz. At this deviation, the tones are attenuated so far that they are nearly impossible to distinguish. The bit error rate would likely approach 50% as noise is the predominant cause of energy.

Finally, the decimation rate for this module is 20x. Because the input sample rate is 8MHz complex, this results in a complex output of 400kHz.

### Quadrature Demod
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/QuadDemod.PNG)
This block is responsible for translating changes in frequency into representations in complex numbers. It takes the conjugate of a time delayed sample X[n-1] and multiplies it by the current sample x[n]. This math can be seen below:

$'(a[n]+ib[n])(a[n-1]-ib[n-1]'$

$'a[n]a[n-1]-ia[n]b[n-1]+ib[n]a[n-1]+b[n]b[n-1]'$

This gives the change between these two values. The output is then the angle of this complex number, ie the angular velocity between these two samples in radians. Then, the gain coefficent is multiplied to the resulting output to create a floating point value.

Because the signal has been basebanded in the previous module, we know that a positive frequency corresponds to a 1 and a negative frequency corresponds to a 0. This information is critical for the next section.

### Binary Slicer
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/BinarySlicer.PNG)

The binary slicer accepts floating point input and outputs a binary value based on whether the input was positive or negative. In the previous section, we used the quadrature demod to show negative frequencies as being negative floating point values and positive frequencies as being positive floating point values. This module converts those positive and negative values into 1's and 0's, thus completing the demod.

### UChar to float
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/Char_to_float.PNG)
This module converts the binary input into a floating point value. The function only serves to allow for it to be used on the QT GUI functions for visual purposes. The demodulation is complete at binary slicer.

### Demodulation GUI
This sections shows what happens when the frequency deviation leaves the expected bandwidth. It uses a deviation of 3kHz and 100kHz to demonstrate how noise becomes the predominant part of the signal when out of band of the filter.

#### Inband Signal
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/inBand_Demod.PNG)

The above figure shows a frequency deviation of 3kHz. The frequency translations can be seen clearly moving between negative and positive values to correspond with the bits which the symbols represent. These transitions are clear and easily disinguishable.

#### Out of Band Signal
![alt text](https://github.com/Ryankearns9/DigComm_Lab3/blob/main/img/outband_Demod.PNG)

In the above figure, it is clear that noise is predominant. The floating point values are extremely small and oscillating between extremely small negative and positive values. Occasionally, these small values are interuppted by random spurious high energy transitions. These spurs are likely the sidelobes of the frequency transitions occuring at inter-symbol periods. This signal is too noisy to get any clear data from. 

