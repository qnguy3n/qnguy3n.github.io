---
layout: page
title: Fractals EEG Motor Imagery
description: Investigating fractal dimension as a non-linear correlate of event-related desynchronization in motor imagery EEG
img: assets/img/fractals-eeg/fig-results-timefreq-fd.png
importance: 6
category: work
toc:
  sidebar: left
_styles: >
  .container {
    max-width: 60% !important;
  }
---

Published at the 37th Annual International Conference of the IEEE Engineering in Medicine and Biology Society (EMBC 2015) {% cite Nguyen2015-nl --file references %}.

This project was carried out during a short-term cultural exchange program with the [Tokyo University of Agriculture and Technology (TUAT)](https://www.tuat.ac.jp/en/), where I was a visiting student in the [Biologically-inspired Computing Laboratory](https://www.livingsyslab.org/pukiwiki/?FrontPage) of Professor Toshiyuki Kondo.

---

## Introduction

Conventional EEG analysis of motor imagery relies on spectral features: specifically, event-related desynchronization (ERD) and event-related synchronization (ERS), which track changes in band-limited power during imagined movement. These are well-established, linear measures that capture the oscillatory dynamics of the sensorimotor cortex. For the background on ERD/ERS and the motor imagery BCI paradigm, see the companion primer [[Primer] Brain-Computer Interfaces Using EEG](/blog/2019/primer-bci-eeg/).

A parallel line of inquiry treats the EEG signal as a non-linear, complex system and asks what its geometric and dynamical properties reveal about brain state. **Fractal dimension** is one such measure: it quantifies the complexity and irregularity of a time series in a way that is orthogonal to spectral power. For the technical background on fractal dimension and Higuchi's algorithm used here, see the companion primer [[Primer] Fractal Dimension in EEG Signal Analysis](/blog/2015/primer-fractal-dimension/).

This study asks a focused question: do the non-linear complexity characteristics of EEG signals, quantified by fractal dimension, covary with ERD during motor imagery? If so, how tightly do they track each other in time, and does this relationship generalize across subjects and datasets?

<figure style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-gammasys-eeg-cap.jpg" alt="GAMMAAsys EEG active electrode system" style="width:80%">
  <figcaption><b>Figure 1:</b> The g.GAMMAAsys active electrode system, representative of high-density EEG acquisition hardware. The dataset used in this study was collected at the Institute for Biomedical Engineering, University of Technology Graz.</figcaption>
</figure>

---

## Methods

### Dataset

The dataset was provided by the Department of Medical Informatics, Institute for Biomedical Engineering, University of Technology Graz, Austria, as part of the BCI Competition III {% cite BCICompIII --file references %}. Three datasets were used:

- **O3VR:** collected from a virtual reality experiment
- **S4b** and **X11b:** collected using an adaptive basket paradigm

<figure style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-vr-feedback.png" alt="Virtual reality feedback paradigm" style="width:60%">
  <figcaption><b>Figure 2:</b> The virtual reality feedback paradigm used in the O3VR dataset. Three frames show the progression of a trial: fixation, cue onset, and feedback with cursor control.</figcaption>
</figure>

Each dataset consisted of two bipolar EEG channels: **C3** and **C4**, placed over the left and right sensorimotor cortices respectively.

<figure style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-electrode-placement.png" alt="EEG electrode placement" style="width:40%">
  <figcaption><b>Figure 3:</b> Bipolar electrode placement used in the Graz datasets. Two bipolar channels centered on C3 and C4 were derived from three electrode pairs on each side, 5 cm apart.</figcaption>
</figure>

The experiment consisted of 3 sessions per subject, each with 9 runs of 40 feedback trials, for a total of 1080 trials per subject. Within each trial, a fixation cross was shown from 0 to 2 seconds, a motor imagery cue appeared at 2 seconds, feedback ran from approximately 4 to 7.5 seconds, and a pause period followed.

<figure style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-trial-timeline.png" alt="Experiment trial timeline" style="width:75%">
  <figcaption><b>Figure 4:</b> Trial structure. Motor imagery is cued at 2 seconds. The feedback period and classifier output run from approximately 4 to 7.5 seconds.</figcaption>
</figure>

<figure style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-eeg-signal-traces.jpg" alt="Raw EEG signal traces" style="width:70%">
  <figcaption><b>Figure 5:</b> Representative multi-channel EEG signal traces from a single trial, illustrating the broadband signal characteristics used as input to both the ERD/ERS and fractal dimension analyses.</figcaption>
</figure>

### ERD/ERS Quantification

ERD and ERS were quantified using the standard Pfurtscheller and Aranibar method {% cite Pfurtscheller1979 --file references %}, expressed as a percentage power change relative to a pre-cue baseline interval. Positive values indicate ERS (power increase); negative values indicate ERD (power decrease):

$$\text{ERD/ERS}_j = \frac{P_j - R}{R} \times 100\%$$

where $$P_j$$ is the inter-trial average power at sample $$j$$, and $$R$$ is the mean baseline power over the pre-event reference interval. The method requires a reference period: the signal is first narrow-bandpass filtered (e.g., 5-10 Hz for the lower mu band), squared sample-by-sample to obtain instantaneous power, and averaged across trials before applying the formula. Two bands were analyzed: **5-10 Hz** (lower mu band) and **20-25 Hz** (beta band).

<figure style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-erd-ers-quantification.png" alt="ERD/ERS quantification example" style="width:55%">
  <figcaption><b>Figure 6:</b> ERD/ERS time course (normalized power relative to baseline = 1.0). The blue shaded region shows ERD (power decrease below baseline); the red shaded region shows ERS (power rebound above baseline) after the motor event.</figcaption>
</figure>

### Fractal Dimension Estimation

Fractal dimension was estimated using **Higuchi's algorithm** {% cite Higuchi1988 --file references %}. For a time series $$X(1), \ldots, X(N)$$, define a family of sub-series at interval $$k$$ with initial offset $$m$$:

$$X_k^m = \left\{ X(m + ik) \right\}_{i=0}^{\lfloor(N-m)/k\rfloor}$$

The normalized length of each sub-series is:

$$L_m(k) = \frac{1}{k} \left[ \left( \sum_{i=1}^{\lfloor(N-m)/k\rfloor} \left| X(m+ik) - X(m+(i-1)k) \right| \right) \cdot \frac{N-1}{\lfloor(N-m)/k\rfloor \cdot k} \right]$$

Averaging $$L_m(k)$$ over all offsets $$m = 1, \ldots, k$$ yields $$\langle L(k) \rangle$$. For a fractal signal, this scales as a power law in $$k$$:

$$\langle L(k) \rangle \propto k^{-D}$$

The fractal dimension $$D$$ is the negative slope of the log-log regression of $$\langle L(k) \rangle$$ against $$k$$. Unlike ERD/ERS, the algorithm operates directly on the time-domain signal and requires no baseline period, making it an absolute measure of signal complexity in any given window. A sliding window was applied to produce a FD time course directly comparable to the ERD/ERS spectral-temporal map.

### Comparing the Two Methods

Both ERD/ERS and Higuchi's FD are feature extraction methods targeting the same neural event, but they approach it from fundamentally different angles:

<figure>
<table style="border-collapse:collapse; width:100%">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th style="width:22%">Property</th>
      <th style="text-align:center">ERD/ERS</th>
      <th style="text-align:center">Higuchi's FD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>What it measures</b></td>
      <td style="text-align:center">Relative change in band-limited power</td>
      <td style="text-align:center">Complexity / irregularity of the time-domain signal</td>
    </tr>
    <tr>
      <td><b>Signal model</b></td>
      <td style="text-align:center">Linear (spectral power)</td>
      <td style="text-align:center">Non-linear (scaling law)</td>
    </tr>
    <tr>
      <td><b>Frequency specificity</b></td>
      <td style="text-align:center">Band-specific: requires bandpass filtering first</td>
      <td style="text-align:center">Broadband aggregate; no frequency transform needed</td>
    </tr>
    <tr>
      <td><b>Baseline dependence</b></td>
      <td style="text-align:center">Mandatory: relative to a reference period $$R$$</td>
      <td style="text-align:center">None: $$D$$ is absolute within any window</td>
    </tr>
    <tr>
      <td><b>Sensitivity</b></td>
      <td style="text-align:center">Oscillatory synchronization in a defined band</td>
      <td style="text-align:center">Signal structure across all timescales</td>
    </tr>
    <tr style="border-bottom:2px solid">
      <td><b>Real-time suitability</b></td>
      <td style="text-align:center">Moderate: FFT or filtering pipeline required</td>
      <td style="text-align:center">High: operates on raw samples, no spectral transform</td>
    </tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 1:</b> Comparison of ERD/ERS and Higuchi's fractal dimension as EEG feature extraction methods.</figcaption>
</figure>

A practical consequence of these differences is that ERD/ERS is well-suited to verifying that a subject is correctly performing the motor imagery task, since power suppression in the mu and beta bands is the canonical marker of sensorimotor cortical engagement. Higuchi's FD, on the other hand, does not require a known baseline, generalizes across frequency bands, and tends to capture hidden dynamical structure that linear power metrics can miss. This makes it attractive as an input feature for BCI classifiers, where non-linear complexity measures can complement spectral features and improve robustness.

One further advantage of HFD that is particularly relevant for real-time BCI applications is **computational cost**. HFD operates entirely in the time domain: the algorithm computes absolute differences between samples and sums them, followed by a single linear regression on a log-log plot. These are elementary arithmetic operations, and the overall complexity scales as $$O(N)$$ in the number of samples $$N$$ per window.

ERD/ERS quantification requires first isolating a frequency band, either by computing a Fast Fourier Transform ($$O(N \log N)$$) or by applying a digital bandpass filter (FIR or IIR). Filtering involves repeated convolutions, which are substantially heavier than the additions and subtractions in HFD. This difference compounds in a real-time BCI loop where features must be recomputed every few hundred milliseconds across multiple channels simultaneously.

In short, HFD extracts a meaningful neural feature with less compute and no spectral transform, which is why it is a natural candidate for low-latency, embedded, or resource-constrained BCI systems.

---

## Results

The main result is shown below: for all six channels across the three datasets, the global minimum in FD over time corresponds to the peak ERD event in the spectral-temporal map.

<figure id="fig-results" style="text-align:center">
  <img src="/assets/img/fractals-eeg/fig-results-timefreq-fd.png" alt="Time-frequency maps and fractal dimension" style="width:100%">
  <figcaption><b>Figure 7:</b> Time-frequency maps (top row) and fractal dimension time courses for the 5-10 Hz band (middle row) and 20-25 Hz band (bottom row), for all six channel-dataset combinations (O3VR-C3, O3VR-C4, S4b-C3, S4b-C4, X11b-C3, X11b-C4). Blue areas in the spectral map indicate ERD; red areas indicate ERS. In every case, the global minimum of FD aligns with the ERD peak.</figcaption>
</figure>

The correspondence between the ERD peak (maximum power suppression) and the FD minimum (lowest complexity) held consistently across datasets:

<figure>
<table style="border-collapse:collapse; width:100%; text-align:center">
  <thead>
    <tr style="border-top:2px solid; border-bottom:2px solid">
      <th style="text-align:center">Subject</th>
      <th style="text-align:center">Channel</th>
      <th style="text-align:center">Hand Imagery</th>
      <th style="text-align:center">ERD latency (s)</th>
      <th style="text-align:center">FD latency (s)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="4"><b>O3VR</b></td>
      <td rowspan="2">C3</td>
      <td>Left</td>
      <td>4.92</td>
      <td>4.21</td>
    </tr>
    <tr>
      <td>Right</td>
      <td>4.63</td>
      <td>3.85</td>
    </tr>
    <tr>
      <td rowspan="2">C4</td>
      <td>Left</td>
      <td>4.59</td>
      <td>3.97</td>
    </tr>
    <tr>
      <td>Right</td>
      <td>5.50</td>
      <td>4.33</td>
    </tr>
    <tr>
      <td rowspan="4"><b>S4b</b></td>
      <td rowspan="2">C3</td>
      <td>Left</td>
      <td>6.26</td>
      <td>5.65</td>
    </tr>
    <tr>
      <td>Right</td>
      <td>6.87</td>
      <td>5.65</td>
    </tr>
    <tr>
      <td rowspan="2">C4</td>
      <td>Left</td>
      <td>6.21</td>
      <td>5.65</td>
    </tr>
    <tr>
      <td>Right</td>
      <td>6.28</td>
      <td>5.89</td>
    </tr>
    <tr>
      <td rowspan="4"><b>X11b</b></td>
      <td rowspan="2">C3</td>
      <td>Left</td>
      <td>6.64</td>
      <td>6.85</td>
    </tr>
    <tr>
      <td>Right</td>
      <td>7.43</td>
      <td>7.81</td>
    </tr>
    <tr>
      <td rowspan="2">C4</td>
      <td>Left</td>
      <td>7.63</td>
      <td>6.73</td>
    </tr>
    <tr>
      <td>Right</td>
      <td>6.78</td>
      <td>5.65</td>
    </tr>
    <tr style="border-top:1px solid">
      <td colspan="3"><b>Mean</b></td>
      <td>6.145</td>
      <td>5.52</td>
    </tr>
    <tr style="border-bottom:2px solid">
      <td colspan="3"><b>SD</b></td>
      <td>1.03</td>
      <td>1.24</td>
    </tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 2:</b> Latency of minimum ERD and FD in 5-10 Hz band across all channels and tasks.</figcaption>
</figure>

The alignment is consistent with the interpretation that both measures track the same underlying neural event: the onset and peak of motor cortical engagement during motor imagery. The FD minimum corresponds to a moment of reduced signal complexity, coinciding with the maximal suppression of the mu rhythm.

---

## Conclusion

This study establishes a consistent relationship between ERD patterns and the fractal complexity of EEG during motor imagery:

1. **The global minimum in fractal dimension aligns with the ERD peak** across all datasets and channels examined. This suggests that the two measures are tracking the same underlying neural transition.
2. **When the brain enters an active motor imagery state**, EEG signals become less random and more structured in a particular way, reflected as decreased fractal dimension. This is consistent with the interpretation that motor cortical activity during imagery is not random but organized around specific neural dynamics.
3. **Low FD values during motor imagery** may serve as a strong indicator for the occurrence of motor imagery, offering a non-linear complement to spectral ERD features for BCI classification.

A limitation of this work is that the relationship between fractal dimension and ERS, the post-movement power rebound, was not fully characterized. The FD time course during the ERS period showed less consistent behavior, and the mechanisms governing FD during cortical synchronization remain an open question.

These findings motivate further use of non-linear complexity measures alongside conventional spectral features in motor imagery BCI systems, where they may provide complementary information that improves classification robustness.
