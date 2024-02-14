# SNR_Tool_Development
## Work in progress  
Developed by Wilfried Beslin and Mike Adams  
Maritimes Team Whale
-------------------
Goal: Developing a SNR tool for use in cetacean research

## Introduction
This tool is for use in calculating the signal to noise (SNR) for cetacean vocalizations obtained using JASCO's PAMLAB annotation software.
Initial development was focused on Blue Whale audible calls. The tool currently exists as a helper script with two underlying functions. The helper script (`Blue_SNR_Tool`) imports the JASCO's PAMLAB annotation files, extracts the needed inputs, and then matches the annotated calls to and imports the appropriate .wav files. These inputs are passed to the two underlying functions, the first (`snr.extractSN`) to extract the data within the .wav files that corresponds to the annotated call and a sample of noise taken some time before the call. These clippings are bandpassed to the frequencies of interest using a *insert filter type here*. The clipped and bandpassed call and noise samples are then passed to the second function (`snr.calculateSNR`) to calculate the SNR value. The final SNR value is then appended to JASCO's PAMLAB annotation dataframe.  

## Set up

### Initial Start Up
1) Either clone or download the .zip and unzip the SNR_Tool_Development repository to your local machine.
2) Add this new directory with all of its subfolders to your MATLAB path.
### Requirements

The tool requires inputs to calculate the SNR values. These include:
  -  SNR_PARAMS.csv: a parameter file which contains the filtering and noise presets for each specie's call type. This file has values for:
      - Species - The species of interest (e.g. Blue Whale)
      - Call Type - The call type (e.g. Tonal) 
      - Lower Frequency - Lower bound of the bandpass filter (Hz)
      - Upper Frequency - Upper bound of the bandpass filter (Hz)
      - Noise Distance - Value to determine how far before the **signal** the **noise** sample will be taken
      - BP_Buffer - Value to add a buffer to the bandpass filter to minimize edge effects
      - Units - Definition of the units used in Noise Distance and BP_Buffer (seconds or samples)
- A directory containing the .csv annotations output generated by JASCO's PAMLAB. These files contain the annotation output of the manually selected calls. The inputs used from these files are:
  - Filename - used to identify the original .wav file containing the call
  - Relative Start Time - The manually selected start time in seconds of the call annotation relative to the start of the .wav file
  - StartTime90 - The start in seconds from the Relative Start Time of the span of time which contains 90% of the energy within the annotation.
  - StopTime90 -  The end in seconds from the Relative Start Time of the span of time which contains 90% of the energy within the annotation.
- A directory containing all the .wav files for which PAMLAB annotation .csv files exist. *Note: This directory can contain additional .wav files, but **must** contain all .wav files for which PAMLAB annotations exist*   

## Working outline for SNR functions

### **snr.extractSN**
```matlab
[xSignal, xNoise] = snr.extractSN(x, fs, sigStart, sigStop, noiseDist, units)
```
**Purpose**
Extract a matrix of samples from an acoustic timeseries coresponding to a defined start and stop time. Additionally, extract a matrix of samples of the same length some defined distance before the signal of interest, to represent a sample of background noise. The interval seperating the signal samples and the noise samples is user defined and dependant on the acoustic properties of the signal. *The signal and noise matrices will potentially be bandpass filtered within this function.* 
**Inputs**
- x = data vector
- fs = sampling rate
- sigStart = signal start time or sample
- sigStop = signal stop time or sample
- noiseDist = distance from signal from which to sample noise, in time or samples
- units = string specifying if start, stop, and distance inputs represent time or samples

**Outputs**
- xSignal = extracted signal samples
- xNoise = extracted noise samples

### **snr.calculateSNR**
```matlab
[snrVal] = calculateSNR(xSignal, xNoise)
```
**Purpose**
Calculate the signal to noise ratio given pre-isolated windows of signal and noise. 
**Inputs**
- xSignal = signal samples
- xNoise = noise samples

**Outputs**
- snrVal = Signal to Noise Ratio value (dB)