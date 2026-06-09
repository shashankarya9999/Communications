# Real-Time Modeling of Analog Modulation Schemes using Software Defined Radio (SDR)

This repository contains the software architectures, digital signal processing (DSP) flowgraphs, and hardware-in-the-loop (HITL) deployment configurations for real-time generation, transmission, and validation of legacy analog modulation schemes. 

Developed as a specialized research assignment under the supervision of the Head of Department (HOD) at the **Indian Institute of Engineering Science and Technology (IIEST), Shibpur**, this project bridges theoretical communication engineering with physical, over-the-air (OTA) RF validation using Software-Defined Radios (SDRs) and laboratory-grade test equipment.

The project validates theoretical signal processing models over-the-air (OTA) using Universal Software Radio Peripheral (USRP) hardware, with empirical signal verification conducted via a Vector Signal Generator (VSG) and Vector Signal Analyzer (VSA).

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
*   **Mathematical Concept:** $s(t) = [A_c + m(t)] \cos(2\pi f_c t)$
*   **DSP Pipeline:** Baseband audio/sine signal source $\rightarrow$ DC Offset insertion (to ensure $1 + \mu \cdot m_n(t) > 0$ for envelope detection) $\rightarrow$ Multiplier block with Local Oscillator (LO) source $\rightarrow$ Low-Pass Filter $\rightarrow$ USRP Sink.
*   **Hardware Test Focus:** Analysis of power distribution efficiency between the carrier and sidebands. Evaluation of distortion effects when the modulation index $\mu > 1$ (over-modulation) using the VSA.

### 2. Double Sideband Suppressed Carrier (DSB-SC)
*   **Mathematical Concept:** $s(t) = m(t) \cdot \cos(2\pi f_c t)$
*   **DSP Pipeline:** Source Signal $\rightarrow$ Balanced Mixer (Multiplier) with a pure Carrier Sinusoid $\rightarrow$ USRP Sink.
*   **Hardware Test Focus:** Verification of carrier suppression depth (measured in dBc below the sidebands) on the Spectrum Analyzer to confirm mixer balance.

### 3. Single Sideband Suppressed Carrier (SSB-SC)
*   **Mathematical Concept:** $s(t) = m(t)\cos(2\pi f_c t) \mp \hat{m}(t)\sin(2\pi f_c t)$
*   **DSP Pipeline:** Implemented using the **Filter Method** and the **Phase Shift (Hilbert Transform) Method**.
    *   *Hilbert Method:* Message signal split $\rightarrow$ Branch A passed directly to an in-phase mixer; Branch B passed through a `Hilbert Transform` block (creating a $90^\circ$ phase shift) to a quadrature-phase mixer $\rightarrow$ Output summed/subtracted to isolate the Upper Sideband (USB) or Lower Sideband (LSB).
*   **Hardware Test Focus:** Measurement of unwanted sideband rejection ratio and tracking carrier leakage under real-world I/Q phase imbalances.

### 4. Narrowband Frequency Modulation (NBFM)
*   **Mathematical Concept:** $s(t) = A_c \cos(2\pi f_c t + \beta \sin(2\pi f_m t))$ where modulation index $\beta \le 0.5$.
*   **DSP Pipeline:** Audio Source $\rightarrow$ Integrator Block $\rightarrow$ Phase Modulator or direct feed into an active VCO configuration limited to narrow deviation bounds ($< 5\text{ kHz}$ bandwidth limits).
*   **Hardware Test Focus:** Analysis of Carson's Rule approximations for bandwidth and checking the emergence of first-order sidebands on the VSA.

## RF Analysis & Measurement Framework

Using the VSA and Spectrum Analyzer, the physical transmitted waveforms were subjected to rigorous validation metrics:

*   **Occupied Bandwidth (OBW):** Measured at $99\%$ signal energy to ensure tight spectral confinement matching theoretical metrics ($2B$ for DSB, $B$ for SSB).
*   **Carrier Suppression Ratio:** Quantified the suppression depth in DSB-SC modes, targetting $> 40\text{ dB}$ attenuation relative to the peak power of the intelligence sidebands.
*   **Harmonic Distortion:** Monitored the RF output spectrum for extraneous intermodulation products arising from DAC non-linearities or over-driving the USRP's internal Tx Gain stages.
