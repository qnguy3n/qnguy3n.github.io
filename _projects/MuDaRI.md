---
layout: page
title: MuDaRI
description: Multilabel Dataset of Retinal Images for Detection of Multiple Ocular Diseases
img: assets/img/MuDaRI/data_aug.png
importance: 3
category: work
related_publications: true
toc:
  sidebar: left
_styles: >
  .container {
    max-width: 60% !important;
  }
---

This project was presented at the 4th [Asia Pacific Tele-Ophthalmology Society (APTOS) Symposium 2019](https://2019.asiateleophth.org), Chennai, India. [[Slides]](https://drive.google.com/file/d/1xU4NCehv5JpsWe5GjpTsvY2rQV8vt3yw/view)

This work also received **Best Poster Presentation** at the Southeast Asia Machine Learning School (SEA MLS) 2019, Jakarta, Indonesia.

## Introduction

Automated screening of retinal fundus photographs has made significant progress over the past decade, driven largely by the availability of large, well-annotated datasets. However, most publicly available datasets, such as EyePACS {% cite Cuadros2009 --file references %}, are organized around a single disease category, meaning each image carries only one diagnostic label, typically for diabetic retinopathy (DR), age-related macular degeneration (AMD), or glaucoma. This single-label paradigm creates a critical gap when deployed in real clinical settings: a model trained exclusively on DR, for instance, can only report whether DR is present or absent, leaving every other concurrent pathology undetected. The clinical consequence is an uncomfortable and potentially misleading outcome where a screening tool reports a patient is clear of one condition while remaining entirely silent about others, as illustrated in [**Figure 1**](#fig-eyepacs).

<figure id="fig-eyepacs" style="text-align:center">
  <img src="/assets/img/MuDaRI/intro-eyepacs.png" alt="Illustration of the single-label limitation of existing datasets such as EyePACS" style="width:90%">
  <figcaption><b>Figure 1:</b> Existing large-scale retinal datasets such as EyePACS are organized around a single diagnostic label per image, making them unsuitable for multi-disease screening in clinical practice.</figcaption>
</figure>

Real patients frequently present with co-occurring conditions: a diabetic patient may simultaneously exhibit DR, macular edema, and early cataract changes within the same fundus image. Addressing this gap requires datasets that are both broader in disease coverage and structured to support multi-label annotation, where a single image can carry several concurrent diagnoses.

In this project, we curate one of the first multilabel datasets of retinal fundus images covering a wide range of ocular diseases, and we develop a multilabel classifier trained on this data to perform simultaneous detection of multiple conditions from a single image.

## Building the Dataset

### Curation Pipeline

The dataset was sourced from Cao Thang International Eye Hospital (Ho Chi Minh City, Vietnam), a private tertiary eye care center, covering a six-year collection period from 2013 to 2018. A total of 6,871 fundus photographs were retrieved and matched against the corresponding Electronic Medical Records (EMRs). All diagnostic terms were standardized and verified to be in compliance with ICD-10 coding conventions, and the dataset was fully de-identified prior to use.

Rather than relying on a purely manual labeling workflow, we integrated convolutional neural network (CNN)-based tools at intermediate stages to assist with image quality filtering and preliminary label assignment, which were then reviewed and corrected by human graders. This hybrid approach, combining automated pre-screening with expert verification, allowed us to scale the curation process without sacrificing annotation quality. The full pipeline is illustrated in [**Figure 2**](#fig-pipeline-dataset).

<figure id="fig-pipeline-dataset" style="text-align:center">
  <img src="/assets/img/MuDaRI/method-building-dataset.png" alt="Dataset curation pipeline" style="width:100%">
  <figcaption><b>Figure 2:</b> Overview of the dataset curation pipeline. CNN-based tools assist with quality filtering and initial labeling at each stage, with human expert review at critical checkpoints.</figcaption>
</figure>

### Dataset Composition

The final curated dataset spans eight major diagnostic categories, with a long tail of less common conditions grouped under an "Others" class. **Table 1** summarizes the disease distribution, and [**Figure 3**](#fig-dataset-dist) shows the resulting label distribution.

<figure id="tab-disease-counts">
<table style="border-collapse:collapse; width:50%; margin-left:auto; margin-right:auto">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th>Condition</th>
      <th style="text-align:right">Occurrence</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Cataract</td><td style="text-align:right">2,258</td></tr>
    <tr><td>Diabetic Retinopathy</td><td style="text-align:right">1,085</td></tr>
    <tr><td>Glaucoma</td><td style="text-align:right">812</td></tr>
    <tr><td>Macular Edema</td><td style="text-align:right">760</td></tr>
    <tr><td>Macular Degeneration</td><td style="text-align:right">757</td></tr>
    <tr><td>Retinal Vascular Occlusion</td><td style="text-align:right">592</td></tr>
    <tr><td>Optic Neuritis/Neuropathy</td><td style="text-align:right">580</td></tr>
    <tr style="border-bottom:2px solid"><td>Others</td><td style="text-align:right">2,199</td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 1:</b> Disease occurrence counts in the full MuDaRI dataset.</figcaption>
</figure>

The "Others" category consolidates all conditions with fewer than 500 occurrences, including Posterior Uveitis, Eye Infections, Macular Pucker, Vitreous/Retinal Hemorrhage, Posterior Capsular Opacification, Retinal Detachment/Breaks, Laser Scars, Hereditary Retinal Dystrophy, Myopia, Central Serous Chorioretinopathy, Glaucoma Suspect, Hypertensive Retinopathy, Chorioretinal Atrophy, Large Cup, and Chorioretinal Neovascularization.

Because images can carry multiple simultaneous diagnoses, the dataset yields 59 unique combinations of disease labels. This combinatorial diversity is precisely what makes MuDaRI challenging and valuable: no two patients are guaranteed to present identically, and a model must learn to assign any subset of the eight classes independently rather than selecting a single winner.

<figure id="fig-dataset-dist" style="text-align:center">
  <img src="/assets/img/MuDaRI/result-data.png" alt="Distribution of disease labels in MuDaRI" style="width:90%">
  <figcaption><b>Figure 3:</b> Label distribution across the MuDaRI dataset, illustrating the class imbalance present in real-world retinal screening data.</figcaption>
</figure>

## Multilabel Classifier

### Data Splits

Images were drawn from two sources: the CTEH collection and the Messidor dataset. **Table 2** summarizes the data sources, and **Table 3** shows the per-condition breakdown across train, validation, and test splits.

<figure id="tab-sources">
<table style="border-collapse:collapse; width:70%; margin-left:auto; margin-right:auto">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th>Dataset</th>
      <th>Partition</th>
      <th style="text-align:right">Images</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>CTEH</td><td>Train</td><td style="text-align:right">5,351</td></tr>
    <tr><td>CTEH</td><td>Test</td><td style="text-align:right">592</td></tr>
    <tr><td>Messidor</td><td>Train</td><td style="text-align:right">988</td></tr>
    <tr><td>Messidor</td><td>Test</td><td style="text-align:right">113</td></tr>
    <tr style="border-bottom:2px solid"><td><strong>Total</strong></td><td></td><td style="text-align:right"><strong>7,048</strong></td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 2:</b> Data sources and partition sizes.</figcaption>
</figure>

<figure id="tab-splits">
<table style="border-collapse:collapse; width:70%; margin-left:auto; margin-right:auto">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th>Condition</th>
      <th style="text-align:right">Train</th>
      <th style="text-align:right">Validation</th>
      <th style="text-align:right">Test</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Cataract</td><td style="text-align:right">1,823</td><td style="text-align:right">202</td><td style="text-align:right">233</td></tr>
    <tr><td>Diabetic Retinopathy</td><td style="text-align:right">1,353</td><td style="text-align:right">150</td><td style="text-align:right">168</td></tr>
    <tr><td>Glaucoma</td><td style="text-align:right">662</td><td style="text-align:right">74</td><td style="text-align:right">76</td></tr>
    <tr><td>Macular Edema</td><td style="text-align:right">765</td><td style="text-align:right">85</td><td style="text-align:right">104</td></tr>
    <tr><td>Macular Degeneration</td><td style="text-align:right">508</td><td style="text-align:right">68</td><td style="text-align:right">81</td></tr>
    <tr><td>Retinal Vascular Occlusion</td><td style="text-align:right">468</td><td style="text-align:right">52</td><td style="text-align:right">72</td></tr>
    <tr><td>Optic Neuritis/Neuropathy</td><td style="text-align:right">470</td><td style="text-align:right">52</td><td style="text-align:right">58</td></tr>
    <tr><td>Others</td><td style="text-align:right">1,800</td><td style="text-align:right">200</td><td style="text-align:right">199</td></tr>
    <tr style="border-bottom:2px solid"><td>Normal</td><td style="text-align:right">415</td><td style="text-align:right">46</td><td style="text-align:right">54</td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 3:</b> Per-condition image counts across train, validation, and test splits.</figcaption>
</figure>

### Model Architecture

Our classifier is built on EfficientNet-B3 {% cite Tan2019 --file references %}, a highly parameter-efficient convolutional network that achieves strong ImageNet performance via compound scaling of depth, width, and resolution. We initialize the network with ImageNet pretrained weights and replace the final classification head with a 9-node fully connected layer with sigmoid activations, enabling independent probability estimates for each disease class. This formulation treats the problem as nine simultaneous binary classification tasks rather than a single mutually exclusive choice, directly accommodating images with multiple concurrent diagnoses. The adapted architecture is shown in [**Figure 4**](#fig-arch).

<figure id="fig-arch" style="text-align:center">
  <img src="/assets/img/MuDaRI/efficientnet-b3-architecture.jpg" alt="EfficientNet-B3 architecture adapted for multilabel classification" style="width:100%">
  <figcaption><b>Figure 4:</b> EfficientNet-B3 architecture with the final layer replaced by a 9-node sigmoid head for multilabel disease classification.</figcaption>
</figure>

Two additional techniques were applied to improve generalization. First, real-time data augmentation (random flips, rotations, color jitter, and zoom) was applied during training to expand the effective diversity of the training set and reduce overfitting. Second, the training loss was modified to account for class imbalance, which is substantial given that conditions such as Glaucoma and Retinal Vascular Occlusion are far less frequent than Cataract. Finally, to support interpretability and facilitate clinician feedback, saliency maps {% cite Simonyan2014 --file references %} are computed for each prediction, highlighting the image regions that most strongly drive the model's output. Examples of augmented images and their saliency maps are shown in [**Figure 5**](#fig-augmentation).

<figure id="fig-augmentation" style="text-align:center">
  <img src="/assets/img/MuDaRI/imaug-salency-map.jpg" alt="Data augmentation examples and saliency maps" style="width:100%">
  <figcaption><b>Figure 5:</b> Left: examples of real-time data augmentation applied during training. Right: saliency maps overlaid on fundus images, highlighting lesion regions that the model attends to for each predicted condition.</figcaption>
</figure>

### Results

The model was evaluated using standard example-based multilabel metrics on the held-out test set, reported in **Table 4**. Per-class confusion matrices are shown in [**Figure 6**](#fig-confusion).

<figure id="tab-results">
<table style="border-collapse:collapse; width:50%; margin-left:auto; margin-right:auto">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th>Metric</th>
      <th style="text-align:right">Score</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Exact Match Ratio</td><td style="text-align:right">0.59</td></tr>
    <tr><td>Accuracy</td><td style="text-align:right">0.76</td></tr>
    <tr><td>Recall</td><td style="text-align:right">0.89</td></tr>
    <tr><td>Specificity</td><td style="text-align:right">0.95</td></tr>
    <tr><td>Precision</td><td style="text-align:right">0.79</td></tr>
    <tr style="border-bottom:2px solid"><td>F1-Score</td><td style="text-align:right">0.82</td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 4:</b> Example-based multilabel classification metrics on the test set.</figcaption>
</figure>

The exact match ratio of 0.59 reflects cases where the model's predicted label set perfectly matches the ground truth, a strict criterion given the combinatorial space of possible multi-disease configurations. The high recall (0.89) and specificity (0.95) indicate that the classifier is conservative in producing false negatives while also avoiding excessive false alarms, which is desirable for a screening context where missing a true positive carries greater clinical cost than a false referral.

<figure id="fig-confusion" style="text-align:center">
  <img src="/assets/img/MuDaRI/confusion-matrix.jpg" alt="Per-class confusion matrices" style="width:90%">
  <figcaption><b>Figure 6:</b> Per-class confusion matrices on the test set. Each matrix shows the true positive, false positive, false negative, and true negative counts for one disease category.</figcaption>
</figure>

### Application Demo

To demonstrate real-world utility, the classifier was integrated into a prototype screening interface. A clinician uploads a fundus photograph and receives a ranked list of predicted conditions alongside the corresponding saliency map, enabling rapid triage and supporting a more transparent second-opinion workflow. Screenshots of the interface are shown in [**Figure 7**](#fig-demo).

<figure id="fig-demo" style="text-align:center">
  <img src="/assets/img/MuDaRI/app-demo1.jpg" alt="Application demo screenshot 1" style="width:80%; margin-bottom:1em">
  <img src="/assets/img/MuDaRI/app-demo2.jpg" alt="Application demo screenshot 2" style="width:80%">
  <figcaption><b>Figure 7:</b> Screenshots of the prototype screening application. The interface displays predicted disease labels with confidence scores and overlays the saliency map on the input fundus image.</figcaption>
</figure>

## Conclusion

This project addresses a structural gap in retinal AI research: the absence of large-scale, multilabel annotated datasets that reflect the co-occurrence of multiple ocular diseases in real clinical populations. By curating MuDaRI and training an EfficientNet-B3-based multilabel classifier on it, we demonstrate that a single model can simultaneously screen for eight major retinal conditions from one fundus photograph, with strong recall and specificity across classes.

The high recall in particular has practical implications for deployment: screening systems must minimize missed diagnoses, and the model's behavior aligns with this priority. The saliency map integration further supports clinical adoption by making the model's reasoning interpretable to ophthalmologists, enabling more productive human-AI collaboration.

Future directions include expanding the dataset with additional disease categories, incorporating multi-reader annotations to quantify labeling uncertainty, and evaluating the classifier across imaging devices and patient populations to assess generalizability.
