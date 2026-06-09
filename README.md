# Real-Time Modeling of Analog Modulation Schemes using Software Defined Radio (SDR)

This repository contains the software architectures, digital signal processing (DSP) flowgraphs, and hardware-in-the-loop (HITL) deployment configurations for real-time generation, transmission, and validation of legacy analog modulation schemes. 

This project validates theoretical signal processing models over-the-air (OTA) using Universal Software Radio Peripheral (USRP) hardware, with empirical signal verification conducted via a Vector Signal Generator (VSG) and Vector Signal Analyzer (VSA).

## System Architecture & Hardware-in-the-Loop Setup

The core objective of this project was to transition from pure mathematical simulation to hardware-validated RF environments. This setup analyzes real-world channel and hardware impairments, including phase noise, local oscillator (LO) leakage, I/Q imbalance, and power amplifier non-linearities.

### Hardware Components
* **SDR Transceiver:** USRP (Universal Software Radio Peripheral)
* **Analysis Equipment:** Vector Signal Analyzer (VSA) for spectral verification, constellation tracking, and Occupied Bandwidth (OBW) measurements.
* **Signal Reference:** Vector Signal Generator (VSG) used to benchmark receiver sensitivity and carrier synchronization.

### Software Environment
* **OS:** Linux (Ubuntu 20.04/22.04 recommended for UHD stability)
* **Framework:** GNU Radio Companion (v3.8 or v3.10)
* **Hardware Driver:** UHD (USRP Hardware Driver)

## 📻 Implemented Modulation Schemes & DSP Architecture

Each scheme was constructed utilizing base mathematical principles mapped to discrete GNU Radio block layouts.

### 1. Double Sideband Full Carrier (DSB-FC)

<img width="2836" height="2084" alt="dsb_fc_USRP_page-0001" src="https://github.com/user-attachments/assets/4d6fc364-a58d-41a9-b75f-6e4f56377dab" /><br>

*   **Mathematical Concept:** $s(t) = [A_c + m(t)] \cos(2\pi f_c t)$
*   **DSP Pipeline:** Baseband audio/sine signal source $\rightarrow$ DC Offset insertion (to ensure $1 + \mu \cdot m_n(t) > 0$ for envelope detection) $\rightarrow$ Multiplier block with Local Oscillator (LO) source $\rightarrow$ Low-Pass Filter $\rightarrow$ USRP Sink.
*   **Hardware Test Focus:** Analysis of power distribution efficiency between the carrier and sidebands. Evaluation of distortion effects when the modulation index $\mu > 1$ (over-modulation) using the VSA.

### 2. Double Sideband Suppressed Carrier (DSB-SC)

<img width="2621" height="1842" alt="dsb_sc_USRP_page-0001" src="https://github.com/user-attachments/assets/89a5d78f-0289-41c2-8356-81529ac96783" /><br>

*   **Mathematical Concept:** $s(t) = m(t) \cdot \cos(2\pi f_c t)$
*   **DSP Pipeline:** Source Signal $\rightarrow$ Balanced Mixer (Multiplier) with a pure Carrier Sinusoid $\rightarrow$ USRP Sink.
*   **Hardware Test Focus:** Verification of carrier suppression depth (measured in dBc below the sidebands) on the Spectrum Analyzer to confirm mixer balance.

### 3. Single Sideband Suppressed Carrier (SSB-SC) 
*   **Mathematical Concept:** $s(t) = m(t)\cos(2\pi f_c t) \mp \hat{m}(t)\sin(2\pi f_c t)$
*   **DSP Pipeline:** Implemented using the **Frequency Discrimination Method** and the **Phase Discrimination (Hilbert Transform) Method**.
   *   **Frequency Discrimination Method:** This architecture generates SSB-SC by first producing a standard DSB-SC signal through a balanced mixer and then passing it through an ultra-sharp, narrow-band bandpass filter to isolate the desired sideband (USB or LSB). Because it relies on physical filtering, it requires high- $Q$ factor filters (such as crystal or ceramic filters) with extremely steep roll-off characteristics to successfully suppress the unwanted sideband without distorting the adjacent message spectrum.

<img width="2730" height="1875" alt="ssb_sc_freq_discrimination_USRP_page-0001" src="https://github.com/user-attachments/assets/666bf003-f053-414f-8b4c-91ccb1bdccf8" /><br>
 
   *   **Phase Discrimination (Hilbert Transform) Method:** This architecture bypasses physical filtering by utilizing two parallel modulation paths fed with exact $90^\circ$ phase-shifted variants of both the carrier and the modulating baseband signals (the latter via a Hilbert Transform). When the outputs of these two branches are summed or subtracted, destructive interference completely cancels out one sideband while constructive interference reinforces the other, generating a clean SSB-SC signal mathematically.

<img width="2000" height="1233" alt="image" src="https://github.com/user-attachments/assets/7e21670f-0045-4755-b183-6af3564648f8" /><br>

*   **Hardware Test Focus:** Measurement of unwanted sideband rejection ratio and tracking carrier leakage under real-world I/Q phase imbalances.

### 4. Narrowband Frequency Modulation (NBFM)
<img width="2000" height="846" alt="image" src="https://github.com/user-attachments/assets/5ede685a-d2a1-4701-9a75-06ba289a05b0" /><br>

*   **Mathematical Concept:** $s(t) = A_c \cos(2\pi f_c t + \beta \sin(2\pi f_m t))$ where modulation index $\beta \le 0.5$.
*   **DSP Pipeline:** Audio Source $\rightarrow$ Integrator Block $\rightarrow$ Phase Modulator or direct feed into an active VCO configuration limited to narrow deviation bounds ($< 5\text{ kHz}$ bandwidth limits).
*   **Hardware Test Focus:** Analysis of Carson's Rule approximations for bandwidth and checking the emergence of first-order sidebands on the VSA.

## RF Analysis & Measurement Framework

Using the VSA and Spectrum Analyzer, the physical transmitted waveforms were subjected to rigorous validation metrics:

*   **Occupied Bandwidth (OBW):** Measured at $99\%$ signal energy to ensure tight spectral confinement matching theoretical metrics ($2B$ for DSB, $B$ for SSB).
*   **Carrier Suppression Ratio:** Quantified the suppression depth in DSB-SC modes, targetting $> 40\text{ dB}$ attenuation relative to the peak power of the intelligence sidebands.
*   **Harmonic Distortion:** Monitored the RF output spectrum for extraneous intermodulation products arising from DAC non-linearities or over-driving the USRP's internal Tx Gain stages.
