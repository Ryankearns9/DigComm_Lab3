# Lab 3

## Introduction
Lab 3's objective is to build an end-to-end FSK Simulation. It includes both the modulation and demodulation portions of the FSK path.

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
The below figure shows the overall flow diagram of the algorithm. It includes the RTL-SDR hardware, resamplers, FM Demod, low pass filter, and audio sink. Additional modules inlcude the GUI functions, GUI variables, and constants for the program. Each will be detailed throughout this section.
![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/FlowDiagram.PNG)

### RTL-SDR Source
The below image shows the RTL-SDR source. The specific hardware is linked above with a full description in Lab 1. This module acts as the ADC and antenna for this module. It basebands the signal of interest with the center frequency being chosen by the GUI variable "center_freq." Further description of this variable is detailed below. The sampling rate is chosen by the constant samp_rate and is set to 2 MHz.

Of note, the Nooelec NESDR SMArt XTR SDR basebands and filters the signal using the E4000 architecture. This prevents interference and aliasing from out of bandwidth signals.

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/RTL_Source.PNG)

The RTL-SDR Source is fed into the first rational resampler 

### Rational Resampler 1
The below image shows the first rational resampler used in the flow chart. It is fed by the RTL SDR source and feeds the low pass filter. A deeper description into rational resampling can be found in Lab 1.

The specific parameters for this module were chosen to be an interpolation rate of 1 and a decimation rate of 5. This decimates the sampling rate to 400 kSps. The FTC puts channel spacing at 200 kHz for FM broadcasting. Decimating to 400 kHz allows an oversampling ratio of 2 which reduces error when demodulating the waveform, thus creating a cleaner signal

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/RationalResampler.PNG)

The first rational resampler feeds the low pass filter

### Low Pass Filter
The low pass filter parameters are described in the below chart. As described in the previous section, the channel size as defined by the FTC is 200kHz. Filtering in this case prevents adjacent channel interference when demodulating the waveform. Without this filter, adjacent channels may leak through to the FM demod creating a much noiser signal.

| Parameter        | Value         |
| ---------------- |:-------------:|
| Fitler Type      | Hamming       |
| Beta             | 6.76          |
| Passband Edge    | 100kHz        |
| Transition Width | 25kHz         |

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/LowPassFilter.PNG)

The low pass filter feeds the FM Demod.

### FM Demod
The FM Demod is a built in GNU Radio function which demodulates FM signals. It accepts 7 inputs which are described in the below table. Of particular importance is the channel rate, deviation, audio pass, audio stop, and tau.

The Channel rate describes the sample rate of the data coming in. This is an important parameter for the algorithm as without it, the system has a bitstream without context. In this case, 400kSps is used which is defined by the resampler 1 described above. This gives an oversampling ratio of 2 for the channel bandwidth.

Deviation describes the difference between the maximum and minimum frequencies compared to the signal carrier. In the United States, broadcast radio uses 75kHz deviation. This allows for a 25kHz worth of gap between the maximum/minimum frequencies and the channel edges.

Audio pas and audio stop describes the audio filtering which is done after the demodulation. The values chosen 15kHz and 16kHz respectively, are chosen because 15 kHz is the upper end of human hearing. The 1 kHz rolloff reduces the filter requirmenets

Tau describes the deemphasis required to reduce the amplitudes of higher frequency signals. In the United States, 75 microseconds is the standard pre-emphasis applied to FM signals. Pre-emphasis is the act of applying gain to high frequency signals through a high pass filter described by RC=75 us. Deemphasis undoes this high pass filter.
While, it is commonly stated that pre-emphasis improves SNR of the signal, this is actually a myth and makes no sense. As described in [this article by Bob Schmid](https://www.repeater-builder.com/rbtip/predeemp.html#:~:text=So%20what%20is%20the%20original%20reason%20for%20Pre%2DEmphasis%3F&text=The%20FM%20broadcasting%20industry%20uses,signal%2Dto%2Dnoise%20ratios.) pre-emphasis is applied for historic reasons. Phase modulation pre-dates frequency modulations. Recievers were already built to handle phase modulation in which pre-emphasis is a byproduct of PM. The pre-emphasis was applied to FM signals to allow PM Recievers to still work with FM modulation. The standard has stuck since those days.

| Parameter        | Value         |
| ---------------- |:-------------:|
| Channel Rate     | 400KHz        |
| Audio Decimation | 1             |
| Deviation        | 75k           |
| Audio Pass       | 15kHz         |
| Audio Stop       | 16kHz         |
| Gain             | 1             |
| Tau              | 75u           |

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/FMDemod.PNG)

The FM Demod feeds into the second resampler

### Rational Resampler 2 
The purpose of the second resampler is to prepare the data stream for output through the speakers. The speaker hardware works at 48 kHz. As a result, the output of the FM modulation must be resampled to allow for connectivity with the speakers. This scales the entire audio by a constant value resulting in the attenuation of output volume. This gives the end user fine-volume control

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/RationalResampler2.PNG)

### Multiply Constant
The multiply constant acts as a volume control. It accepts the GUI Range Variable "volume". It applies a multiplication value between 0 and 1 to the audio output from the frequency modulator. 

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/MultiplyConst.PNG)

The output of the multiply constant is the Audio Sink

### Audio Sink
The audio sink is the hardware speaker for the computer. It works at 48kHz and converts floating point values to voltages which vibrate a diaphram which can be heard as noise. Further details can be found in Lab 1

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/AudioSink.PNG)

### GUI Output Description
The GUI has 3 main plots along with 2 variable controls. The below figure shows the output. The first plot is a waterfall curve and shows the entire spectrum taken at the output of the RTL-SDR. For further details on waterfall plots, reference Lab 1.

The second plot is an FFT of the output of the FM Demod. It shows the Fourier Transform of the audio so you can visualize the frequencies contained within the radio station.

The third plot is an FFT of the RTL-SDR output. It shows an instananeous view of the frequency domain at the output of the ADC.

The two controls are for center frequency and volume. The center frequency control is fed to the RTL-SDR source to update the frequency. In this plot, it is tuned to 93.3 MHz. 

The second control is for volume and it updates the Multiply Constant. It will scale the amplitude of the audio output.

![alt text](https://github.com/Ryankearns9/DigComm_Lab2/blob/main/imgs/SDRGUI.PNG)