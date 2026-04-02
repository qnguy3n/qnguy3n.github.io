---
layout: post
title: "[Primer] Functional Near-Infrared Spectroscopy (fNIRS) for Brain Sensing"
date: 2019-11-25
description: Biophysics, signal quantification, and mental workload applications of fNIRS as a non-invasive brain imaging tool.
tags: primer BCI fNIRS neuroscience optical-imaging
related_posts: false
related_publications: true
toc:
  sidebar: left
---

> This primer provides the fNIRS background for the [BCI Mental Workload](/projects/BCI-Mental-Workload/) project, which used fNIRS over the prefrontal cortex to measure cognitive load during BCI training. See also the companion primer on [EEG-based BCI paradigms](/blog/2019/primer-bci-eeg/).
{: .block-tip}

Functional Near-Infrared Spectroscopy (fNIRS) is a non-invasive neuroimaging technique that infers brain activity by measuring changes in the concentrations of oxygenated and deoxygenated hemoglobin in the cerebral cortex. It occupies a practical middle ground between EEG (high temporal resolution, low spatial resolution) and fMRI (high spatial resolution, but bulky, expensive, and intolerant of movement). fNIRS offers reasonable cortical coverage, genuine portability, and tolerance to motion, making it attractive for real-world and longitudinal cognitive monitoring.

---

## Neurovascular Coupling

The foundation of fNIRS (and of fMRI's BOLD signal) is **neurovascular coupling**: the tight physiological link between neural activity and local cerebral blood flow. When neurons in a cortical region become active, their metabolic demand increases rapidly. The brain responds by locally dilating blood vessels to deliver more oxygenated blood, a process called the **hemodynamic response**. This vasodilation brings more oxyhemoglobin (HbO) into the activated region, and as oxygen is consumed, converts it to deoxyhemoglobin (Hb).

The canonical pattern of cortical activation is therefore:
- **Increase in [HbO]** (more oxygenated blood delivered)
- **Decrease in [Hb]** (oxygen consumed and deoxygenated blood swept away)

fNIRS detects these concentration changes optically by using near-infrared light that can penetrate the scalp and skull to reach the cortex {% cite Villringer1997 --file references %}.

---

## The Near-Infrared Optical Window

Biological tissues are largely opaque to visible light but become relatively transparent in the **near-infrared (NIR) window, roughly 700-900 nm**. Two chromophores in this window are biologically relevant for brain monitoring:

1. **Oxyhemoglobin (HbO):** absorbs more strongly at longer NIR wavelengths (around 830 nm).
2. **Deoxyhemoglobin (Hb):** absorbs more strongly at shorter NIR wavelengths (around 760 nm).

<div class="row mt-3">
  <div class="col-sm-10 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/bci-mwl/background_NIRS.png" class="img-fluid rounded z-depth-1" caption="Absorption spectra of HbO and Hb in the near-infrared optical window (700-900 nm). Their distinct absorption profiles at different wavelengths allow the two chromophores to be separately quantified from a single measurement." %}
  </div>
</div>

By shining NIR light into the scalp at two or more wavelengths and measuring the backscattered light, an fNIRS device can simultaneously estimate changes in both [HbO] and [Hb].

---

## The Beer-Lambert Law

### Standard Form

In a non-scattering medium, the attenuation of light intensity follows the **Beer-Lambert Law** {% cite Delpy1997 --file references %}:

$$A = \log_{10} \frac{I_0}{I} = a \cdot c \cdot d$$

where:
- $$A$$ is light attenuation (optical density)
- $$I_0$$ is incident light intensity, $$I$$ is transmitted intensity
- $$a$$ ($$\mu\text{M}^{-1}\,\text{cm}^{-1}$$) is the molar extinction coefficient of the absorber
- $$c$$ ($$\mu\text{M}$$) is the concentration of the absorber
- $$d$$ (cm) is the geometrical path length

For a medium containing multiple absorbers, the total attenuation is the linear sum:

$$A = [a_1 c_1 + a_2 c_2 + \cdots + a_n c_n] \cdot d$$

### Modified Beer-Lambert Law

Biological tissue is a highly scattering medium. The chaotic refractive-index mismatches at cell boundaries cause photons to travel long, tortuous paths through tissue rather than straight lines. This has two consequences: it greatly increases the effective photon path length, and it introduces an additive scattering loss term $$G$$ that cannot be independently measured.

The **Modified Beer-Lambert Law** {% cite Cope1991 --file references %} accommodates both effects:

$$A = a \cdot c \cdot d \cdot \text{DPF} + G$$

where the **Differential Pathlength Factor (DPF)** is a multiplicative correction accounting for the elongated photon path due to scattering, and $$G$$ is the scattering-dependent loss term.

Because $$G$$ is unknown, absolute concentrations cannot be recovered from a single measurement. However, if $$G$$ does not change during the recording window (a reasonable assumption for short cognitive experiments), the **change** in concentration can be recovered from a measured change in attenuation:

$$\Delta A = \Delta c \cdot a \cdot d \cdot \text{DPF}$$

This is the practical foundation of fNIRS: the modality measures **relative changes** in hemoglobin concentrations rather than absolute values.

---

## Quantifying $$\Delta[\text{HbO}]$$ and $$\Delta[\text{Hb}]$$

Measuring at two or more wavelengths allows solving for the two unknowns simultaneously. At each wavelength $$\lambda$$, the attenuation change is a linear combination of contributions from both chromophores:

$$\Delta A_\lambda = \bigl(\Delta[\text{HbO}] \cdot \alpha_{\text{HbO},\lambda} + \Delta[\text{Hb}] \cdot \alpha_{\text{Hb},\lambda}\bigr) \cdot d \cdot \text{DPF}$$

For a three-wavelength system such as the SHIMADZU FOIRE-3000 (780, 805, and 830 nm), the system of equations is overdetermined and solved via a least-squares fit over the three measured attenuations, yielding:

$$\Delta[\text{HbO}] = -3.6132 \cdot A_{780} + 1.1397 \cdot A_{805} + 3.0153 \cdot A_{830}$$

$$\Delta[\text{Hb}] = 3.7837 \cdot A_{780} - 0.7833 \cdot A_{805} - 2.5679 \cdot A_{830}$$

(units: $$\mu\text{M} \cdot \text{cm}$$; coefficients are device-specific and derived from the known extinction spectra of HbO and Hb at each wavelength.)

From these, two aggregate quantities are defined:

$$\text{Oxygenation} = \Delta[\text{HbO}] - \Delta[\text{Hb}]$$

$$\text{Blood Volume} = \Delta[\text{HbO}] + \Delta[\text{Hb}]$$

**Oxygenation** (sometimes called the cerebral oxygenation index) is the most informative single measure for cognitive studies: positive values indicate cortical activation, negative values indicate inhibition or task disengagement.

---

## fNIRS Devices and Optode Placement

An fNIRS device consists of **sources** (light-emitting optodes, emitting NIR at two or more wavelengths) and **detectors** (light-collecting optodes). Each source-detector pair defines a **channel**, whose signal reflects hemodynamics in the cortical tissue roughly beneath the midpoint between the two optodes.

The standard source-detector separation is **3 cm**, which allows NIR light to reach approximately 1-2 cm below the scalp surface, penetrating into the cortex, while maintaining an acceptable signal-to-noise ratio. Shorter separations (~8 mm) are used as "short channels" to capture only superficial scalp hemodynamics for noise correction.

Optodes are held in position by a flexible cap or headband. A compact **2×3 source-detector array** over the **prefrontal cortex (PFC)** yields 7 channels and is the standard configuration for cognitive load experiments.

### Why the Prefrontal Cortex?

The PFC is the preferred fNIRS monitoring site for mental workload studies for two practical and two functional reasons:

1. **Accessibility:** the forehead is flat, hairless, and provides good optical contact.
2. **Separation from motor cortex:** fNIRS over the PFC does not interfere with EEG over the central motor regions, enabling true multimodal recording.
3. **Executive control:** the PFC coordinates working memory, sustained attention, and cognitive control, all of which scale with task difficulty.
4. **Workload literature:** a large body of literature confirms that PFC oxygenation tracks cognitive load reliably across diverse task types.

---

## Signal Processing

### Pre-processing Pipeline

Raw optical signals require several pre-processing steps before analysis:

1. **Conversion to [HbO] and [Hb]:** apply the Modified Beer-Lambert equations to the raw optical density time series.
2. **Bandpass filtering:** retain frequencies between 0.01 and 0.5 Hz. This removes slow baseline drifts (below 0.01 Hz) and high-frequency physiological noise: cardiac oscillations (~1 Hz), respiratory fluctuations (~0.3 Hz), and motion artifacts.
3. **Baseline correction:** subtract the mean of a pre-stimulus rest period from each trial, so signals reflect changes relative to a common baseline.
4. **Epoch averaging:** for event-related designs, average baseline-corrected trials across repetitions to improve signal-to-noise ratio.

### Summary Statistics and Statistical Analysis

The mean amplitude of the hemodynamic response within the task window provides a scalar feature per trial and condition. These features can then be compared across conditions using one-way ANOVAs, post-hoc tests, or Pearson correlation with behavioral measures (e.g., accuracy, NASA-TLX scores).

---

## Applications in Mental Workload Assessment

**Mental workload (MWL)** is broadly defined as the ratio between task demands and available cognitive capacity. fNIRS over the PFC is well suited to MWL measurement because the PFC is heavily recruited during demanding cognitive tasks (working memory, attention, executive control), and its hemodynamic response is unaffected by motor activity in the sensorimotor cortex. This orthogonality to motor signals is a critical advantage when the primary task itself involves motor control, as in BCI training based on motor imagery.

A large body of literature confirms that PFC oxygenation increases monotonically with cognitive task difficulty across a wide range of paradigms {% cite Girouard2009 Sassaroli2008 Solovey2011 Ayaz2012 --file references %}:
- Arithmetic calculation and n-back working memory tasks
- Attention-demanding visual search
- Air traffic control and flight simulation
- Musical performance under varying complexity

The canonical **hemodynamic response to mental load** is positive $$\Delta[\text{HbO}]$$ and negative $$\Delta[\text{Hb}]$$ over the PFC, growing larger with task difficulty and subsiding once the task ends or the participant mentally disengages. At very high workload levels, a non-monotonic rebound has been reported, interpreted as task disengagement when demands exceed the participant's capacity.

### NASA Task Load Index as Subjective Reference

In MWL studies, physiological measures like fNIRS are typically validated against **subjective self-report**. The standard instrument is the [NASA Task Load Index (NASA-TLX)](https://humansystems.arc.nasa.gov/groups/tlx/) {% cite Hart1988 --file references %}, a six-subscale questionnaire covering:

- Mental Demand
- Physical Demand
- Temporal Demand
- Performance (self-assessed)
- Effort
- Frustration

Each subscale is rated on a 0-100 scale, and the unweighted mean provides an overall workload score. NASA-TLX is widely used as a behavioral reference in neuroimaging studies of cognitive load, but it is retrospective, coarse, and susceptible to individual differences in introspective accuracy. The degree to which physiological signals like fNIRS agree or diverge from NASA-TLX is itself informative about which measure captures more of the true cognitive state.

---

## fNIRS in Hybrid BCI Systems

The independence of fNIRS from EEG at both the sensor and signal level makes it a natural complement in hybrid BCI systems {% cite Yuksel2016 --file references %}. EEG captures the fast, electrical control signal (motor imagery ERD/ERS), while fNIRS monitors the slow, hemodynamic state of the PFC (cognitive load, engagement, fatigue). Together, they can support an adaptive BCI that not only decodes user intent but also monitors whether the user is in a productive cognitive state, adjusting task difficulty or providing rest prompts accordingly. This dual-channel approach is the motivation for the [BCI Mental Workload](/projects/BCI-Mental-Workload/) project.
