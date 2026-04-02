---
layout: page
title: FAZSEG
description: Automatic Foveal Avascular Zone Segmentation Using Hessian-Based Filter and U-Net Deep Learning Network
img: assets/img/FAZSEG/sample_output.png
importance: 4
category: work
related_publications: true
toc:
  sidebar: left
_styles: >
  .container {
    max-width: 60% !important;
  }
---

This work as been published at [8th International Conference on the Development of Biomedical Engineering in Vietnam](https://link.springer.com/chapter/10.1007/978-3-030-75506-5_68) (BME2020). Github's [repo](https://github.com/qnguy3n/FAZSEG)

<!-- [Full text](https://drive.google.com/file/d/1owZMp2b_wBaOWNeudv_cxfOKU_Q8our-/view). -->


## Introduction
Optical Coherence Tomography Angiography (OCTA) has emerged as a rapid, high-resolution, and non-invasive imaging modality for generating volumetric angiographic images. It is a highly promising tool for evaluating and detecting common retinal diseases, including age-related macular degeneration, diabetic macular edema, and choroidal neovascularization {% cite deCarlo2015 Virgili2015 Jia2014 --file references %}. A critical biomarker in assessing these conditions is the foveal avascular zone (FAZ), as its accurate extraction is vital for detecting vascular diseases that affect retinal microcirculation.

Historically, researchers have relied on traditional computer vision algorithms from simple morphological algorithm {% cite Diaz2019 --file references %} to advanced active contours algorithm such as Generalized Gradient Vector Flow (GGVF) {% cite Lu2018 --file references %} or Level Set {% cite Lin2020 --file references %}, to segment the FAZ. However, these conventional techniques often struggle to adapt to complex anatomical variations and image noise. More recently, the field has shifted toward Deep Convolutional Neural Networks (DCNN) to automate FAZ segmentation. While these deep learning approaches show immense promise, their effectiveness and generalization across varied clinical datasets still require further investigation.

To address these limitations, we propose a hybrid pipeline that bridges traditional computer vision techniques with a modern U-Net-based model to automatically and accurately quantify the FAZ in OCTA images. First, we apply a Hessian-based filter to selectively enhance the blood vessels within the OCTA scans. We then feed these enhanced images into a specialized U-shape semantic segmentation model to extract the avascular zone. We demonstrate that this combined approach robustly outperforms traditional baseline methods, such as the Level Set, by a significant margin.

<figure id="fig-pipeline" style="text-align:center">
  <img src="/assets/img/FAZSEG/pipeline.png" alt="Proposed pipeline: raw OCTA scan → Hessian Filter → enhanced image → U-Net → FAZ mask" style="width:100%">
  <figcaption><b>Figure 2:</b> Overview of the proposed pipeline. A raw OCTA scan is first processed by the Hessian-based filter to enhance vessel contrast, then fed into a U-Net model to produce the final FAZ segmentation mask.</figcaption>
</figure>

## Methodology and Materials

### Dataset

Two independent clinical datasets were used in this study. The primary dataset was provided by the Joseph Carroll Lab (Medical College of Wisconsin, Milwaukee, WI, USA) and consists of 230 OCTA images acquired from the superficial capillary plexus. Each image in this dataset was manually delineated by a single expert grader using ImageJ (National Institutes of Health, Bethesda, MD); further details on the acquisition protocol and grading procedure can be found in {% cite Linderman2017 --file references %}. The second dataset was sourced from the Duke Eye Center and comprises 30 OCTA images of the deep vascular complex (DVC), representing a distinct imaging layer with different contrast and morphological characteristics.

The two datasets were split as follows: 184 images from the JC dataset for training, all 30 Duke images for validation, and the remaining 46 JC images held out as the test set. The Duke dataset was designated as the validation set because this configuration consistently yielded superior model performance — as measured by the Jaccard coefficient — compared to the reverse arrangement. Prior to being passed to the network, all images were enhanced by the Hessian-based filter described in the following section.

<figure style="text-align:center">
  <div style="position:relative; padding-bottom:56.25%; height:0; overflow:hidden; max-width:100%">
    <iframe src="https://www.youtube.com/embed/q_FjroTTiWs" title="Manual FAZ grading process in ImageJ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="position:absolute; top:0; left:0; width:100%; height:100%"></iframe>
  </div>
  <figcaption>The manual FAZ grading process performed in ImageJ by an expert grader. Each OCTA image is traced by hand to delineate the foveal avascular zone boundary, producing the ground-truth masks used to train and evaluate the segmentation model.</figcaption>
</figure>

### Hessian Filter

The Hessian filter analyzes the local second-order curvature of an image to detect and enhance tubular structures such as blood vessels, while suppressing blobs and flat backgrounds. We use the Frangi vesselness formulation {% cite Frangi1998 --file references %}, which scores each pixel by how well its local geometry matches a tube-like shape across multiple scales. For a full conceptual and mathematical treatment, see the [[Primer] Hessian Filter for Vessel Enhancement](/blog/2019/primer-hessian-filter/) post.

#### Tuning the Scale - Practical Considerations for $\sigma$
The most critical parameter when deploying the Frangi filter is the Gaussian scale, $\sigma$. Because the algorithm relies on Gaussian smoothing before calculating the Hessian matrix, the value of $\sigma$ directly corresponds to the radius of the tubular structures you intend to extract.

When processing clinical data, in our case, isolating the microvasculature in OCTA scans, configuring $\sigma$ correctly is the difference between a clean extraction and a noisy mess. A small $\sigma$ (e.g., 0.5) sensitizes the filter to fine, delicate capillaries. A larger $\sigma$ (e.g., 2.5) shifts the focus to wider, primary vessels.

In practice, a single $\sigma$ is rarely sufficient. You must define a range ($\sigma_{min}$ to $\sigma_{max}$) that covers the expected anatomical variation in vessel radii within your specific imaging modality. The algorithm computes the vesselness score at multiple scales within this range and retains the maximum response for each pixel. An example of the filter applied to retinal OCT-A data is shown in [**Figure 1**](#fig-hessian).

<figure id="fig-hessian" style="text-align:center">
  <img src="/assets/img/FAZSEG/github_FAZ_filter.png" alt="Hessian filter applied to FAZ data" style="width:75%">
  <figcaption><b>Figure 1:</b> Example of the Hessian-based filter applied to our retinal OCT-A data, with sigma values $\sigma$ range to be from 0.5 to 2.5</figcaption>
</figure>


### U-Net

Our segmentation model builds on the U-Net architecture {% cite Ronneberger2015 --file references %}, a fully convolutional encoder–decoder network originally developed for biomedical image segmentation. The key idea of U-Net is its symmetric structure: a contracting encoder path that progressively captures high-level semantic context, paired with an expanding decoder path that recovers spatial resolution through up-sampling, with skip connections bridging the two halves to preserve fine-grained boundary details.

We depart from the original design by replacing the vanilla encoder with a SE-ResNeXt-50 backbone {% cite Hu2019 --file references %}. SE-ResNeXt-50 combines the grouped convolutions of ResNeXt with Squeeze-and-Excitation (SE) blocks, which recalibrate channel-wise feature responses by explicitly modelling interdependencies between channels. This allows the encoder to produce richer, more discriminative feature maps while remaining computationally tractable. The decoder branch is then redesigned to progressively up-sample these encoded features back to full resolution, producing the final binary FAZ segmentation mask. Although other backbone architectures were explored, SE-ResNeXt-50 consistently yielded the best segmentation performance in our experiments.

The model was trained for 70 epochs with a batch size of 8, input resolution of 256 × 256, a learning rate of 0.0002, and the Adam optimizer.


### Level Set

Originally introduced by Osher and Sethian {% cite Osher1988 --file references %}, the Level Set method has become a widely adopted tool in biomedical image segmentation owing to its capacity to represent and evolve boundaries of complex, irregular shapes. The core idea is to embed the segmentation contour as the zero level set of a higher-dimensional scalar function $\phi$, called the level set function (LSF). Rather than tracking the boundary explicitly, the algorithm evolves $\phi$ according to image-derived forces — typically based on intensity gradients — causing the zero contour to migrate toward and lock onto object edges.

In this study we use the Distance Regularized Level Set Evolution (DRLSE) formulation {% cite Li2010 --file references %}, which augments the standard energy functional with a regularization term that penalizes deviations of $\phi$ from a signed distance function. This eliminates the need for periodic re-initialization — a costly and numerically error-prone step required by classical level set methods — and permits a broader and more robust class of initializations. To automate the method, we develop an intensity-based heuristic to place the initial contour, and then run the DRLSE evolution on Hessian-filtered images to identify the FAZ boundary. For a full conceptual and mathematical treatment, see the [[Primer] Level Set Methods for Image Segmentation](/blog/2019/primer-level-set/) post.

<figure id=”fig-levelset-demo” style=”text-align:center”>
  <video autoplay loop muted playsinline style=”width:60%”>
    <source src=”/assets/img/FAZSEG/demo.mp4” type=”video/mp4”>
  </video>
  <figcaption><b>Figure 3:</b> Level Set evolution on our retinal OCT-A data, showing the contour converging to the FAZ boundary.</figcaption>
</figure>

<figure style=”text-align:center”>
  <div style=”position:relative; padding-bottom:56.25%; height:0; overflow:hidden; max-width:100%”>
    <iframe src=”https://www.youtube.com/embed/amzJiJf-R0I” title=”Level Set method demonstration” frameborder=”0” allow=”accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture” allowfullscreen style=”position:absolute; top:0; left:0; width:100%; height:100%”></iframe>
  </div>
  <figcaption>Demonstration of how the Level Set method works: an initial contour evolves by minimizing an energy functional, expanding or contracting until it converges to the target boundary. The DRLSE formulation used in this project adds a regularization term that keeps the level set function close to a signed distance function, removing the need for re-initialization.</figcaption>
</figure>

### Evaluation
#### Jaccard Score

The Jaccard index (also known as Intersection over Union, IoU) measures the overlap between the predicted segmentation mask $\hat{S}$ and the ground truth mask $S$:

$$J(S, \hat{S}) = \frac{|S \cap \hat{S}|}{|S \cup \hat{S}|} = \frac{TP}{TP + FP + FN}$$

where $TP$ is the number of true positive pixels, $FP$ is false positives, and $FN$ is false negatives. The score ranges from 0 (no overlap) to 1 (perfect agreement). It is stricter than the Dice coefficient because it penalizes both over- and under-segmentation symmetrically.

#### False Positive and False Negative

A **false positive** (FP) occurs when a pixel is predicted as FAZ but does not belong to the ground truth mask — the model over-segments into background or vessel regions. A **false negative** (FN) occurs when a pixel belongs to the ground truth FAZ but is missed by the prediction — the model under-segments the region. Both are normalized by the total ground truth area:

$$FP = \frac{|\hat{S} \setminus S|}{|S|}, \qquad FN = \frac{|S \setminus \hat{S}|}{|S|}$$

A low FP indicates few spurious predictions; a low FN indicates good sensitivity to the full extent of the FAZ. Together with the Jaccard score, they provide a more complete picture of where a method tends to fail.

## Results
Three configurations were evaluated: Hessian Filter + Level Set, U-Net alone, and the proposed Hessian Filter + U-Net pipeline. The first two serve as baselines — the Level Set baseline isolates the contribution of the deep learning component, while the standalone U-Net baseline quantifies the benefit of Hessian preprocessing. Quantitative results are reported in **Table 1**.

<figure id="tab-results">
<table style="border-collapse:collapse; width:100%">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th rowspan="2">Methods</th>
      <th colspan="2" style="text-align:center">Jaccard</th>
      <th colspan="2" style="text-align:center">FP</th>
      <th colspan="2" style="text-align:center">FN</th>
    </tr>
    <tr style="border-bottom:1px solid">
      <th style="text-align:center">Mean</th>
      <th style="text-align:center">STD</th>
      <th style="text-align:center">Mean</th>
      <th style="text-align:center">STD</th>
      <th style="text-align:center">Mean</th>
      <th style="text-align:center">STD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Hessian + Level Set</td>
      <td style="text-align:center">54.59</td>
      <td style="text-align:center">37.12</td>
      <td style="text-align:center">27.37</td>
      <td style="text-align:center">41.53</td>
      <td style="text-align:center">44.59</td>
      <td style="text-align:center">38.64</td>
    </tr>
    <tr>
      <td>Hessian + U-Net</td>
      <td style="text-align:center">87.76</td>
      <td style="text-align:center">5.49</td>
      <td style="text-align:center">3.59</td>
      <td style="text-align:center">5.26</td>
      <td style="text-align:center">8.83</td>
      <td style="text-align:center">4.31</td>
    </tr>
    <tr style="border-bottom:2px solid">
      <td>U-Net</td>
      <td style="text-align:center">86.49</td>
      <td style="text-align:center">9.63</td>
      <td style="text-align:center">9.17</td>
      <td style="text-align:center">9.53</td>
      <td style="text-align:center">5.17</td>
      <td style="text-align:center">4.62</td>
    </tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 1:</b> Segmentation performance (mean ± STD) of each method in terms of Jaccard coefficient, False Positive (FP), and False Negative (FN).</figcaption>
</figure>

The proposed Hessian Filter + U-Net pipeline achieved the highest mean Jaccard score (87.76%), outperforming both the standalone U-Net (86.49%) and the Hessian Filter + Level Set baseline (54.59%) by a wide margin. Beyond raw accuracy, the Hessian preprocessing had a pronounced effect on prediction consistency: the standard deviation of the Jaccard score dropped from 9.63% for U-Net alone to 5.49% with preprocessing, indicating that the filter produces cleaner, more structured input that the network can segment more reliably across diverse cases. The proposed method also recorded the lowest mean FP (3.59%) and the lowest standard deviations for both FP and FN, suggesting that its predictions are not only accurate on average but also tightly concentrated around that average. Representative segmentation outputs are shown in [**Figure 4**](#fig-results).

<figure id="fig-results" style="text-align:center">
  <img src="/assets/img/FAZSEG/output-examples.png" alt="Segmentation output examples" style="width:100%">
  <figcaption><b>Figure 4:</b> Example segmentation outputs for three retinal OCT-A scans (rows). Each row shows the input image with the ground-truth FAZ boundary (red contour), followed by the predicted masks and per-image scores (Jaccard J, FP, FN) for Hessian Filter + Level Set, U-Net, and Hessian Filter + U-Net.</figcaption>
</figure>

The Level Set baseline performed poorly overall, with a mean Jaccard of only 54.59% and an extremely high variance (STD 37.12%), reflecting frequent segmentation failures rather than a consistent but inaccurate result. Two factors likely account for this. First, the low imaging resolution (304 × 304 pixels) combined with heavy intra-FAZ noise makes it difficult for gradient-based contour evolution to distinguish true FAZ boundaries from noise artifacts, causing the contour to stall or collapse prematurely. Second, the initial contour was hard-coded to the image center, which is problematic in practice since the FAZ is not always centered in the field of view. A misplaced initialization can trap the contour in a local energy minimum far from the true boundary, a failure mode that deep learning avoids entirely by learning a global feature representation of the region.

## Conclusion

This study demonstrates that deep convolutional neural networks are a highly effective tool for FAZ segmentation in OCTA images. Even when trained on a relatively small dataset, the SE-ResNeXt-50-based U-Net substantially outperformed the Level Set baseline, a classical technique that remains widely used in clinical image analysis, across all three evaluation metrics. This gap underscores the ability of deep networks to learn robust, data-driven representations that generalize across the anatomical and imaging variability inherent in clinical OCTA data, without requiring hand-tuned initialization or manual parameter selection.

Beyond the core architecture, the results confirm that Hessian-based preprocessing provides a meaningful and consistent benefit. By enhancing the vascular contrast of the input images prior to segmentation, the filter supplies the network with cleaner, more structured input, which translates into higher mean accuracy and, crucially, a tighter distribution of predictions across samples. The reduction in Jaccard standard deviation, from 9.63% for U-Net alone to 5.49% with Hessian preprocessing, suggests that the filter mitigates the effect of low-quality or noisy scans that would otherwise destabilize the model's output.

One aspect that warrants further investigation concerns the choice of validation set. During training, we observed that using the Duke dataset for validation and the JC dataset for testing yielded a noticeably better final model than the reverse split. Because neither the validation nor the test set contributes to gradient updates, this asymmetry is not immediately explained by standard overfitting arguments. One plausible hypothesis is that the Duke and JC datasets differ subtly in image acquisition protocol, contrast characteristics, or FAZ morphology distribution, causing the validation loss landscape to differ between the two splits in ways that affect early stopping and learning rate scheduling. Characterizing these inter-dataset differences more rigorously, and developing validation strategies that are robust to them, remains an important direction for future work.
