---
layout: post
title: "A Practical, Hands-On Introduction to Brain-Computer Interfaces"
date: 2019-12-01
tags: BCI EEG machine-learning signal-processing
---

> This post is a companion to the `Brain Computer Interface (BM074IU)` course at the Biomedical Engineering Department, International University – Vietnam National University, HCMC. All experiments were conducted using a low-cost, portable EEG device.

All code and data are available on GitHub: [qnguy3n/ml-for-bci](https://github.com/qnguy3n/ml-for-bci/)


---

## What is a Brain-Computer Interface?

A Brain-Computer Interface (BCI) is a system that creates a direct communication pathway between the brain and an external device — no muscles involved. You think, and the machine responds.

BCIs work by recording electrical activity from the brain (EEG), processing the signals, and translating patterns into commands. Applications range from assistive technology for paralyzed patients to gaming and neurofeedback.

In this post, we walk through **two real experiments** from scratch:
1. **Alpha waves** — detecting open vs. closed eyes
2. **SSVEP** — controlling a device by looking at flickering stimuli

---

## The Hardware: WEEG

Both experiments use **WEEG**, a portable, low-cost, 4-bit 8-channel EEG recording system. It records EEG in two differential channels at a **sampling rate of 250 Hz**.

Electrodes are placed on the scalp following the **International 10-20 Electrode Placement System** — the standard coordinate system used in EEG research worldwide.

For both experiments, the electrode configuration is:

| Position | Role |
|---|---|
| Left Mastoid | DRL (ground/bias) |
| Right Mastoid | CH1− |
| O2 | CH1+ |
| O1 | CH2+ |
| Oz | CH2− |

Occipital electrodes (O1, O2, Oz) are chosen because they sit over the **visual cortex** — the brain region most active during visual processing.

<div class="row mt-3">
  <div class="col-sm-8 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/posts/bci/electrode-placement.png" class="img-fluid rounded z-depth-1" caption="Electrode placement following the International 10-20 System." %}
  </div>
</div>

---

## Experiment 1: Alpha Waves (Open and Closed Eyes)

### Background

**Alpha waves** are neural oscillations in the **8–12 Hz** frequency band. A well-established finding in neuroscience is that alpha power **increases when you close your eyes** and **decreases when you open them**. This makes the open/close eye paradigm one of the simplest, most reliable BCI experiments you can run.

### Protocol

- The subject (23-year-old, voluntary) sits relaxed on a chair.
- **Phase 1:** Eyes open for 15 seconds (baseline).
- **Phase 2:** Alternates between **eyes closed for 5s** and **eyes open for 5s** per trial.
- Each run has at least 5 trials; 5 runs total per session.

<div class="row mt-3">
  <div class="col-sm-8 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/posts/bci/protocol.png" class="img-fluid rounded z-depth-1" caption="Experiment protocol: 15s baseline, then alternating 5s closed / 5s open eyes." %}
  </div>
</div>

### Signal Processing

#### Step 1 — Load and Visualize

```matlab
addpath '../../data/test alpha 5s'
addpath '../../tools/spectral'

ALLEEG = load('5sopen_close.mat').ALLEEG;   % EEGLAB format
sig = ALLEEG.data;

figure; plot(sig)
```

> **Tip:** The first few hundred samples are usually contaminated by movement artifacts from putting on the headset. Skip them.

```matlab
start = 700;  % skip first ~2.8 seconds
subplot(2,1,1), plot(sig(start:end))
subplot(2,1,2), plot_spectrogram(sig, start, 1, gca)
colormap(jet(256));
```

#### Step 2 — Isolate Events

```matlab
events = ALLEEG.event;
Fs     = ALLEEG.srate;     % 250 Hz
duration = Fs * 5;         % 5-second window

event1 = events(1).latency;  % odd indices = closed, even = open

closed_sample = sig(event1:(event1 + duration));
open_sample   = sig((event1 - duration):event1);

figure;
subplot(2,1,1), plot(closed_sample), title('Eyes closed')
subplot(2,1,2), plot(open_sample),   title('Eyes open')
```

#### Step 3 — Spectral Analysis

```matlab
figure; plotFFT(closed_sample, Fs, gca);
```

You will see a clear **peak around 8–12 Hz** in the closed-eye spectrum — this is your alpha wave signature.

<div class="row mt-3">
  <div class="col-sm-10 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/posts/bci/signal-and-spectrogram.png" class="img-fluid rounded z-depth-1" caption="Raw EEG signal (top) and its spectrogram (bottom). Alpha power visibly increases during closed-eye periods." %}
  </div>
</div>

<div class="row mt-3">
  <div class="col-sm-8 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/posts/bci/spectrogram.png" class="img-fluid rounded z-depth-1" caption="Power Spectral Density: high alpha power (red) during closed eyes, low (blue) during open eyes." %}
  </div>
</div>

### Feature Extraction

For classification, we extract two features from each 1-second window:
- **Feature 1:** Maximum spectral power in the 8–13 Hz band
- **Feature 2:** Signal-to-noise ratio (SNR) in the 8–13 Hz band

```matlab
n_samples  = 100;
n_features = 2;
window     = Fs * 1;     % 1-second window
band       = [6, 35];
params.Fs  = Fs;
params.band = band;

class1_feat = zeros(n_samples, n_features);  % closed eyes
class2_feat = zeros(n_samples, n_features);  % open eyes

for i = 1:n_samples
    % Randomly sample a window from a closed-eye event
    event_chosen = events_close(randi(length(events_close) - 2));
    feat_range   = event_chosen:(event_chosen + duration - window);
    feat_start   = feat_range(randi(numel(feat_range)));
    feat_window  = sig(feat_start:(feat_start + window - 1));

    class1_feat(i, :) = feature_extraction(feat_window, params);
end

% Repeat for open eyes (class2_feat) ...
```

### Classification

Once features are extracted, we apply **logistic regression** (implemented from scratch):

```matlab
X = cat(1, x1, x2);                             % stack both classes
y = cat(1, zeros(n_samples,1), ones(n_samples,1));

w = rand(n_features, 1);   % initialize weights
b = rand();

% Gradient descent
alpha   = 1;
num_iter = 10;

for iter = 1:num_iter
    z = w' * X' + b;
    g = 1 ./ (1 + exp(-z));    % sigmoid

    J      = -mean(y .* log(g') + (1 - y) .* log(1 - g'));   % cross-entropy loss
    grad_w = (1/n_samples) * X' * (g' - y);
    grad_b = mean(g' - y);

    w = w - alpha * grad_w;
    b = b - alpha * grad_b;
end
```

**Result:** The two feature clusters (open vs. closed) are linearly separable, and even a simple logistic regression achieves high accuracy.

---

## Experiment 2: SSVEP

### Background

**Steady-State Visual Evoked Potentials (SSVEP)** are another powerful BCI paradigm. When you look at a flickering stimulus at a fixed frequency (e.g., 10 Hz), your visual cortex resonates at **that exact frequency** — producing a measurable peak in the EEG spectrum.

By presenting multiple stimuli flickering at different frequencies simultaneously, a user can "select" a target simply by **looking at it**. No training required.

### Protocol

- Subject (26-year-old) sits and stares at a flickering LCD screen.
- Stimuli alternate between **stimulus on (10s)** and **off (10s)** for four trials per run.
- Each trial uses one of four target frequencies: **6.6 Hz, 7.5 Hz, 8.75 Hz, 10 Hz**.
- The subject looks at each frequency for its trial in order.

<div class="row mt-3">
  <div class="col-sm-10 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/posts/bci/ssvep-protocol.png" class="img-fluid rounded z-depth-1" caption="SSVEP protocol: each trial alternates 10s stimulus on / 10s off at a target frequency." %}
  </div>
</div>

Same electrode placement as Experiment 1 (Oz, O1, O2 over the visual cortex).

### Signal Processing

#### Pre-processing

Same filtering pipeline as alpha waves:
- **High-pass at 0.3 Hz** — remove DC drift
- **Low-pass at 43 Hz** — remove high-frequency noise

#### Spectrogram Analysis

```matlab
% Run ssvep_signal_processing.m
plot_spectrogram(sig, start, 1, gca);
colormap(jet(256));
```

In the spectrogram, you will see **long red stripes** (high power) at exactly 6.6, 7.5, 8.75, and 10 Hz — each aligned to the time window when the subject was looking at that frequency. This is the SSVEP response.

<div class="row mt-3">
  <div class="col-sm-10 mt-3 mt-md-0 mx-auto">
    {% include figure.liquid path="assets/img/posts/bci/spectrogram.png" class="img-fluid rounded z-depth-1" caption="SSVEP spectrogram (Channel Oz): long red stripes at target frequencies indicate evoked brain responses." %}
  </div>
</div>

#### Feature Extraction

```matlab
% feature_extraction.m
% Extract peak power at target frequencies: 6.6, 7.5, 8.75, 10 Hz
% and their harmonics (2x, 3x the fundamental)
```

The dominant feature is simply **which target frequency has the highest peak power** in a given window — that frequency is what the user was looking at.

### The SSVEP Game

To make this interactive, we built **FindPrincess** — a simple maze game controlled entirely by SSVEP. Each direction of movement corresponds to one flickering frequency. The subject navigates the maze by looking at the matching stimulus.

```
6.6 Hz → Left
7.5 Hz → Right
8.75 Hz → Up
10 Hz  → Down
```

Source code is in `tools/SSVEP_BCIGame/FindPrincess.py`.

---

## Key Takeaways

| | Alpha Waves | SSVEP |
|---|---|---|
| **Paradigm** | Internal state (eyes open/closed) | Exogenous (visual stimulus) |
| **Frequency band** | 8–12 Hz | Stimulus frequency + harmonics |
| **Training needed** | Yes (calibration run) | Minimal |
| **Use case** | Fatigue/drowsiness detection | Navigation, selection |
| **Complexity** | Low | Low–Medium |

Both paradigms demonstrate the core BCI pipeline:

**Record → Filter → Extract Features → Classify → Act**


---

*Material developed for BM074IU — Brain Computer Interface, International University, VNU-HCMC, Fall 2019.*
