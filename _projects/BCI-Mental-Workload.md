---
layout: page
title: BCI Mental Workload
description: Monitoring mental workload during sensorimotor rhythm BCI training using functional near-infrared spectroscopy over the prefrontal cortex
img: assets/img/bci-mwl/experiment_system_blockdiagram.png
importance: 5
category: work
related_publications: true
toc:
  sidebar: left
_styles: >
  .container {
    max-width: 60% !important;
  }
---

Bachelor's Thesis, Biomedical Engineering School, International University, Vietnam National University HCMC, 2019.

## Introduction

Sensorimotor rhythm (SMR)-based brain-computer interfaces (BCIs) give paralyzed users direct control of external devices by translating imagined hand movements into machine commands {% cite Yuan2014 --file references %}. The technology is scientifically mature enough to navigate wheelchairs, move robotic arms, and restore cursor control {% cite Leeb2013 --file references %}. Yet it comes with a significant practical burden: users must undergo weeks or months of intensive training to learn to modulate their own EEG signals reliably, and a meaningful fraction of participants drop out because they cannot gain control at all {% cite Myrden2015 --file references %}.

A central but underappreciated problem in this training is **mental workload**. Several reviews have noted that sustaining and manipulating motor imagery can be extremely demanding, especially for disabled users {% cite Yuan2014 --file references %}. Yet most attempts to quantify this burden have relied on post-hoc subjective surveys {% cite Felton2012 --file references %}, which are coarse, retrospective, and unsuitable for real-time adaptive systems. Ideally, a training environment would monitor the user's cognitive state continuously and adapt task difficulty to keep the user in an optimal engagement zone, neither bored at levels below their skill nor overwhelmed above it, following the Yerkes-Dodson curve.

Achieving that requires an objective, real-time measure of mental workload that does not interfere with the EEG control signal. As discussed in the [[Primer] BCI Using EEG](/blog/2019/primer-bci-eeg/) companion post, the EEG features used for motor imagery classification (mu and beta band power modulations, ERD/ERS) overlap substantially with the EEG features used for workload assessment (alpha and theta power changes), making it impractical to use EEG for both purposes simultaneously on the same recording.

This project investigates whether **functional near-infrared spectroscopy (fNIRS)** over the **prefrontal cortex (PFC)** can serve as that workload monitor {% cite Girouard2009 Sassaroli2008 Solovey2011 --file references %}. Because fNIRS measures hemodynamic changes via optical absorption, it is spectrally and spatially orthogonal to the EEG motor signals, making it a natural complementary modality {% cite Villringer1997 --file references %}. For the underlying biophysics and signal processing of fNIRS, see the [[Primer] fNIRS for Brain Sensing](/blog/2019/primer-bci-fnirs/) companion post.

To my knowledge, this is the first study to evaluate fNIRS for mental workload monitoring specifically during SMR-based BCI training.

---

## Methods

### Participants

**Study 1** (calibration stage): three healthy male volunteers, age 20, all naive to BCI.

**Study 2** (adaptive feedback stage): five volunteers (one female, mean age 20.4, SD 1.3 years). One participant was excluded due to excessive fNIRS noise from an unusual forehead geometry that allowed optical leakage. The remaining four were divided by BCI expertise: three naive participants with no prior BCI experience, and one expert participant who had undergone months of prior BCI training and could control external devices by thought alone.

All participants were right-handed (Edinburgh Handedness Inventory) with no reported neurological, cardiovascular, or visual abnormalities. Written informed consent was obtained and the Declaration of Helsinki was followed.

### Dual EEG and fNIRS Recording

EEG and fNIRS were recorded simultaneously throughout all sessions.

**EEG** was acquired with a Biosemi ActiveTwo system. Seventeen electrodes (FCz, FC1-4, Cz, C1-6, CPz, CP1-5) were placed over the primary motor cortex following the 10-20 international system.

**fNIRS** was recorded with a SHIMADZU FOIRE-3000 system at three wavelengths (780, 805, and 830 nm). A 2×3 optode array with 3-cm source-detector separation was mounted over the PFC, yielding 7 channels sampled at 18 Hz. The array was positioned so that the bottom of the headgear touched the tops of the subject's eyebrows, with the central probes aligned to the midline.

<figure id="fig-system" style="text-align:center">
  <img src="/assets/img/bci-mwl/experiment_system_blockdiagram.png" alt="System block diagram" style="width:55%">
  <figcaption><b>Figure 1:</b> Block diagram of the dual recording system. EEG captures the BCI control signal from sensorimotor cortex; fNIRS simultaneously monitors hemodynamic changes over the prefrontal cortex.</figcaption>
</figure>

<figure style="text-align:center">
  <div style="display:flex; justify-content:center; gap:1rem; flex-wrap:wrap; align-items:flex-start">
    <div>
      <img src="/assets/img/bci-mwl/experiment_fnirs_7ch.jpg" alt="fNIRS 7-channel array" style="width:260px">
      <figcaption style="font-size:0.85em">7-channel fNIRS optode array</figcaption>
    </div>
    <div>
      <img src="/assets/img/bci-mwl/experiment_fnirs_real.jpg" alt="fNIRS real setting" style="width:260px">
      <figcaption style="font-size:0.85em">fNIRS headgear during experiment</figcaption>
    </div>
  </div>
  <figcaption><b>Figure 2:</b> The fNIRS optode array placed over the prefrontal cortex.</figcaption>
</figure>

### BCI Software: BCI2000

BCI training sessions were driven by [BCI2000](https://www.bci2000.org/) {% cite BCI2000 --file references %}, an open-source general-purpose BCI platform. BCI2000 acquired the EEG stream in real time, performed online spectral analysis, and rendered the visual feedback interface. After the calibration stage, the frequency bands and electrodes that best separated the two motor tasks were identified from spectral analysis and coefficient-of-determination ($$r^2$$) maps, and these parameters were used to set the linear classifier driving cursor movement in the feedback sessions.

<figure style="text-align:center">
  <img src="/assets/img/bci-mwl/experiment_bci2000.png" alt="BCI2000 interface" style="width:55%">
  <figcaption><b>Figure 3:</b> BCI2000 graphical interface during BCI training, showing the cursor-control feedback task.</figcaption>
</figure>

### Experiment Procedure

#### Study 1: Calibration Stage

Subjects performed **motor execution** (actual hand gripping) and **motor imagery** (imagined hand gripping) in alternating runs. Each session lasted 16 minutes (8 runs of 120 seconds). Within each run, the subject rested for 30 seconds, responded to left/right movement cues over 60 seconds, then rested again. fNIRS was recorded throughout to characterize the PFC hemodynamic response to the two motor tasks before any feedback was introduced.

<figure style="text-align:center">
  <img src="/assets/img/bci-mwl/experiment_run_timescheme.png" alt="Calibration run timeline" style="width:60%">
  <figcaption><b>Figure 4:</b> Timeline of a single calibration run. The subject rests, responds to a motor cue, then returns to rest.</figcaption>
</figure>

#### Study 2: Adaptive Feedback Stage

After calibration, subjects used imagined hand movement to control the vertical position of a cursor moving horizontally across the screen. A target appeared on the right side of the screen, and the subject had to guide the cursor to hit it.

<figure style="text-align:center">
  <img src="/assets/img/bci-mwl/experiment_feedback_protocol.png" alt="Feedback task protocol" style="width:65%">
  <figcaption><b>Figure 5:</b> Timeline of one feedback trial followed by a 30-second rest period. Each run lasted 120 seconds, with roughly 40 trials per run.</figcaption>
</figure>

Task difficulty was manipulated by the number of possible target positions per trial:

| Level | Possible targets | Approximate chance level |
|---|---|---|
| 1 | 3 | 33% |
| 2 | 5 | 20% |
| 3 | 8 | 12.5% |

More targets means a smaller vertical window per target, requiring finer cursor control and, presumably, greater cognitive effort. Each subject completed 4 sessions spaced 5-7 days apart. After each session, subjects completed the **NASA Task Load Index (NASA-TLX)** {% cite Hart1988 --file references %} to provide subjective workload ratings.

### fNIRS Signal Processing

Raw optical signals were converted to $$\Delta[\text{HbO}]$$ and $$\Delta[\text{Hb}]$$ using the Modified Beer-Lambert Law {% cite Villringer1997 Cope1991 --file references %} with device-specific extinction coefficients (see the [fNIRS primer](/blog/2019/primer-bci-fnirs/) for the full formulation). A non-recursive bandpass filter (0.01-0.5 Hz) removed cardiac, respiratory, and motion artifacts. Each trial was baseline-corrected by zeroing its initial value.

In Study 2, only **oxygenation** ($$\Delta[\text{HbO}] - \Delta[\text{Hb}]$$) was used as the primary dependent variable due to the higher dimensionality of the data (multiple subjects, sessions, and difficulty levels).

---

## Results

### Study 1: Motor Execution vs. Motor Imagery

The first study tested whether fNIRS over the PFC could distinguish motor execution from motor imagery in the static calibration setting, before any performance feedback was introduced.

Oxygenation change showed **no significant difference between the two tasks** across all subjects and sessions. The [HbO] and [Hb] signals for both tasks remained low and closely intertwined, making the two conditions indistinguishable.

<figure id="fig-memi" style="text-align:center">
  <img src="/assets/img/bci-mwl/results_MEMI.png" alt="Oxygenation: motor imagery vs. motor execution" style="width:55%">
  <figcaption><b>Figure 6:</b> Mean oxygenation for motor imagery (I) and motor execution (E) tasks across all subjects. No significant difference was found between the two conditions.</figcaption>
</figure>

This indicates that simple motor imagery, in the absence of feedback or performance pressure, does not generate measurable differential mental workload at the PFC level. Study 2 therefore introduced the more cognitively demanding adaptive feedback condition.

---

### Study 2: Adaptive Feedback Control

#### Subjective Workload (NASA-TLX)

NASA-TLX Mental Demand scores confirmed that subjects experienced the three difficulty levels as distinct (F = 35.58, p < 0.001):

| Level | Mean Mental Demand | Std |
|---|---|---|
| 1 (3 targets) | 30.0 | 13.6 |
| 2 (5 targets) | 59.4 | 11.8 |
| 3 (8 targets) | 85.3 | 16.7 |

<figure id="fig-nasatlx" style="text-align:center">
  <img src="/assets/img/bci-mwl/results_nstlx_study2.png" alt="NASA-TLX across difficulty levels" style="width:50%">
  <figcaption><b>Figure 7:</b> NASA-TLX Mental Demand scores across the three difficulty levels (F = 35.58, p < 0.001). The monotonic increase confirms participants perceived the levels as progressively demanding.</figcaption>
</figure>

#### Performance

Accuracy decreased significantly as the number of possible targets increased (F = 18.378, p < 0.001). However, absolute accuracy was close to or only marginally above chance in all conditions, reflecting the early stage of training.

| Level | Absolute Accuracy (%) | Chance Level (%) | Relative Accuracy |
|---|---|---|---|
| 1 | 36.85 | 33.3 | 3.52 |
| 2 | 22.32 | 20.0 | 2.32 |
| 3 | 16.38 | 12.5 | 3.88 |

<figure style="text-align:center">
  <div style="display:flex; justify-content:center; gap:1rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_acc.png" alt="Accuracy across levels" style="width:44%">
    <img src="/assets/img/bci-mwl/results_acc_relation_nstlx.png" alt="Accuracy vs NASA-TLX" style="width:44%">
  </div>
  <figcaption><b>Figure 8:</b> (Left) Absolute accuracy across three difficulty levels. (Right) Moderate negative linear relationship between accuracy and NASA-TLX Mental Demand (Pearson's r = -0.66, p < 0.001).</figcaption>
</figure>

Notably, while NASA-TLX scores tracked difficulty level monotonically, they could not meaningfully discriminate actual performance. This near-chance performance across all levels is expected at the very early stage of motor imagery training, which typically requires a minimum of a dozen sessions before reliable control emerges.

#### fNIRS Hemodynamic Signals

The group-averaged hemodynamic response across all subjects, sessions, and trials revealed two notable patterns.

**Inverted hemodynamic trend.** The dominant pattern was a decrease in [HbO] (rather than the canonical increase) and approximately unchanged [Hb], resulting in consistently negative oxygenation across all task conditions. This inverted response, while atypical relative to the cognitive workload literature (which usually observes positive $$\Delta[\text{HbO}]$$), has been reported before and is associated with a negative BOLD response in fMRI. Critically, all four subjects manifested this pattern, ruling out individual outlier effects.

<figure id="fig-hemo" style="text-align:center">
  <div style="display:flex; justify-content:center; gap:0.5rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_hbo.png" alt="HbO change" style="width:30%">
    <img src="/assets/img/bci-mwl/results_hb.png" alt="Hb change" style="width:30%">
    <img src="/assets/img/bci-mwl/results_oxy.png" alt="Oxygenation change" style="width:30%">
  </div>
  <figcaption><b>Figure 9:</b> Group-averaged hemodynamic changes across the three difficulty levels. [HbO] decreases during all task levels, [Hb] remains approximately flat, and oxygenation is consistently negative, reflecting a uniformly inverted hemodynamic response.</figcaption>
</figure>

Mean oxygenation values across difficulty levels:

| Level | Mean Oxygenation | Std |
|---|---|---|
| 1 | -0.0012 | 0.00601 |
| 2 | -0.0041 | 0.00667 |
| 3 | -0.0015 | 0.00673 |

**Non-monotonic response at peak difficulty.** Oxygenation was most negative at Level 2, with a partial rebound at Level 3. The typical finding in the MWL literature is a monotonic relationship between difficulty and hemodynamic response. The rebound at Level 3 is consistent with findings by Izzetoglu et al. {% cite Izzetoglu2004 --file references %} in air warfare simulations, where oxygenation dropped at the most demanding condition due to participant disengagement once the task exceeded their performance ceiling. We hypothesize the same mechanism applies here: subjects mentally disengaged at Level 3 because its difficulty was far beyond their current skill level at this early training stage.

<figure style="text-align:center">
  <img src="/assets/img/bci-mwl/results_oxy_subs.png" alt="Oxygenation by subject" style="width:80%">
  <figcaption><b>Figure 10:</b> Oxygenation change by individual subject. All four subjects display the same inverted hemodynamic pattern, confirming it is a consistent group-level effect.</figcaption>
</figure>

#### Analysis Across Sessions

To investigate whether fNIRS tracks learning over time, I analyzed oxygenation changes, accuracy, and NASA-TLX scores grouped by difficulty level across the three feedback sessions.

A significant change in oxygenation was found across sessions for **Level 1 only** (F = 6.1195, p < 0.01). Levels 2 and 3 showed no significant session effect. Accuracy followed a similar pattern: Level 1 improved from session 1 (29%, below the 33% chance level) to session 2 (40.3%), then stabilized. NASA-TLX scores, by contrast, remained essentially flat across all sessions and difficulty levels.

<figure style="text-align:center">
  <div style="display:flex; justify-content:center; gap:0.5rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_oxy_ss.png" alt="Oxygenation across sessions" style="width:45%">
    <img src="/assets/img/bci-mwl/results_acc_ss.png" alt="Accuracy across sessions" style="width:45%">
  </div>
  <figcaption><b>Figure 11:</b> Oxygenation (left) and accuracy (right) across the three training sessions, stratified by difficulty level. Only Level 1 shows significant change over time in both measures.</figcaption>
</figure>

<figure style="text-align:center">
  <img src="/assets/img/bci-mwl/results_nstlx_ss.png" alt="NASA-TLX across sessions" style="width:55%">
  <figcaption><b>Figure 12:</b> NASA-TLX Mental Demand scores across sessions. Unlike fNIRS oxygenation, subjective ratings remained flat across all sessions and difficulty levels, indicating that self-report was insensitive to the learning captured by fNIRS.</figcaption>
</figure>

This is the sharpest finding in the study: **fNIRS detected early-stage learning that NASA-TLX could not**. In Level 1, oxygenation became less negative across sessions as subjects gained control, mirroring the accuracy improvement. Subjective workload ratings showed none of this.

Accuracy table across sessions:

| Session | Level 1 Mean (%) | Level 2 Mean (%) | Level 3 Mean (%) |
|---|---|---|---|
| 1 | 29.00 | 20.75 | 17.25 |
| 2 | 40.33 | 24.63 | 12.44 |
| 3 | 41.22 | 21.67 | 19.56 |

#### Individual Analysis: Naive Subjects

Examining subjects individually confirmed a consistent pattern across all three naive participants: oxygenation dropped from Level 1 to Level 2, then partially rebounded at Level 3. Although the omnibus one-way ANOVA was not significant across subjects, targeted one-tailed t-tests based on the hypothesized disengagement pattern were significant for Subject 3 (p < 0.05).

<figure id="fig-individual" style="text-align:center">
  <div style="display:flex; justify-content:center; gap:0.5rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_sub1_difflvl.png" style="width:30%">
    <img src="/assets/img/bci-mwl/results_sub2_difflvl.png" style="width:30%">
    <img src="/assets/img/bci-mwl/results_sub3_difflvl.png" style="width:30%">
  </div>
  <figcaption><b>Figure 13:</b> Oxygenation across difficulty levels for each naive subject. All three display the same non-monotonic pattern: deepest response at Level 2, partial rebound at Level 3.</figcaption>
</figure>

The interaction between difficulty level and session was also consistent: within Level 1, oxygenation change was significant across sessions for all naive subjects (reflecting learning), while Levels 2 and 3 remained stable.

<figure style="text-align:center">
  <div style="display:flex; justify-content:center; gap:0.5rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_sub1_difflvl_time.png" style="width:30%">
    <img src="/assets/img/bci-mwl/results_sub2_difflvl_time.png" style="width:30%">
    <img src="/assets/img/bci-mwl/results_sub3_difflvl_time.png" style="width:30%">
  </div>
  <figcaption><b>Figure 14:</b> Oxygenation across difficulty levels and sessions per naive subject. The Level 1 oxygenation change is significant across sessions for all three subjects, while Levels 2 and 3 remain stable.</figcaption>
</figure>

#### Individual Analysis: Expert Subject

The expert participant presented a strikingly different profile. Performance was significantly higher across all difficulty levels compared to naive subjects (p < 0.05 in all conditions).

<figure style="text-align:center">
  <div style="display:flex; justify-content:center; gap:0.5rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_expert_accuracy.png" style="width:45%">
    <img src="/assets/img/bci-mwl/results_expert_difflvl.png" style="width:45%">
  </div>
  <figcaption><b>Figure 15:</b> (Left) Expert vs. naive accuracy across difficulty levels: expert performance was significantly higher in all three conditions (p < 0.05). (Right) Expert fNIRS oxygenation was uniformly low and did not differ significantly across levels, consistent with task automaticity.</figcaption>
</figure>

Despite superior performance, the expert's fNIRS signals across all difficulty levels were low and did not differ significantly between conditions. In contrast, the expert's NASA-TLX Mental Demand scores were only marginally lower than those of naive participants, a gap disproportionately small relative to the performance difference.

<figure style="text-align:center">
  <div style="display:flex; justify-content:center; gap:0.5rem; flex-wrap:wrap">
    <img src="/assets/img/bci-mwl/results_expert_mentalDemand.png" style="width:45%">
    <img src="/assets/img/bci-mwl/results_individual_oxyvalue.png" style="width:45%">
  </div>
  <figcaption><b>Figure 16:</b> (Left) NASA-TLX Mental Demand: the expert rated tasks only slightly less demanding than naive participants, a modest gap relative to the large performance difference. (Right) Summary of mean oxygenation values across all subjects and difficulty levels.</figcaption>
</figure>

This pattern is consistent with **task automaticity**: after extensive training, the brain's executive control resources are less heavily recruited, enabling superior performance at lower metabolic cost in the PFC. Importantly, fNIRS captured this expertise-related distinction far more clearly than NASA-TLX.

---

## Conclusion

This study provides the first evidence that fNIRS over the prefrontal cortex is responsive to the mental demands of SMR-based BCI training, tracking differences across difficulty levels, learning across sessions, and expertise, in ways that subjective self-report cannot. The key findings are:

1. **Motor imagery alone does not generate detectable PFC workload differences** relative to motor execution, in the static calibration setting without feedback.
2. **Adaptive feedback BCI training induces PFC oxygenation changes** that track task difficulty and participant engagement, with a non-monotonic rebound at peak difficulty consistent with task disengagement.
3. **fNIRS detected early-stage learning** (oxygenation normalization at Level 1 across sessions) that NASA-TLX did not, suggesting higher sensitivity to the neural correlates of motor skill acquisition.
4. **Expert BCI users show uniformly low PFC activity** across difficulty levels, consistent with task automaticity, while subjective ratings showed only marginal separation from naive users.

The results support fNIRS as a complementary sensing modality in future hybrid BCI systems that adapt training difficulty online based on the user's real-time cognitive state, moving beyond current approaches that rely on subjective report or performance alone.
