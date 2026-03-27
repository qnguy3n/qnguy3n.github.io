---
layout: post
title: "[Primer] Hessian Filter for Vessel Enhancement"
date: 2019-11-15
description: An introduction to Hessian-based filtering and the Frangi vesselness measure for tubular structure detection in medical images.
tags: primer CV medical-imaging
related_publications: true
related_posts: false
---

> This post explains the Hessian filter as used in the [FAZSEG](/projects/FAZSEG/) project, which applies it to enhance retinal microvasculature in OCT-A images before segmenting the Foveal Avascular Zone. See the project page for the specific parameters and results on real clinical data.
{: .block-tip}

Hessian filtering is a feature extraction technique that analyzes the local second-order structure of an image. By calculating the second derivatives of image intensities, it determines the local curvature at every pixel. This makes it incredibly powerful for identifying continuous, tubular structures like blood vessels while ignoring uniform backgrounds or unstructured noise.

## The Formulation

To calculate the local geometry, the image is first smoothed using a Gaussian kernel at a specific scale $\sigma$. The Hessian matrix $H$ is then constructed using the second-order partial derivatives of this smoothed image. For a 2D image $I(x,y)$, the Hessian matrix at scale $\sigma$ is defined as:

$$H_\sigma = \begin{bmatrix} L_{xx} & L_{xy} \\ L_{yx} & L_{yy} \end{bmatrix}$$

Here, $L_{xx}$, $L_{xy}$, and $L_{yy}$ represent the convolution of the image with the corresponding second derivatives of the Gaussian kernel.

The core of the filter relies on extracting the eigenvalues ($\lambda_1$ and $\lambda_2$, ordered such that $\lvert\lambda_1\rvert \le \lvert\lambda_2\rvert$) from this matrix. These eigenvalues represent the principal curvatures of the local image topography:

- $\lambda_1$ (Small curvature): Represents the direction along the vessel, where the intensity changes very little.
- $\lambda_2$ (Large curvature): Represents the direction across the vessel, where the intensity changes rapidly from the background to the vessel and back.

## The Frangi Vesselness Measure

The most widely used formulation utilizing the Hessian matrix for this purpose is the Frangi filter {% cite Frangi1998 --file references %}.

For a bright vessel on a dark background (where we expect $\lambda_2 < 0$), the 2D "vesselness" measure $V(\sigma)$ is computed as:

$$V(\sigma) = \begin{cases} 0 & \text{if } \lambda_2 > 0 \\ \exp\left(-\frac{R_B^2}{2\beta^2}\right) \left(1 - \exp\left(-\frac{S^2}{2c^2}\right)\right) & \text{otherwise} \end{cases}$$

Where:

- $R_B = \frac{\lvert\lambda_1\rvert}{\lvert\lambda_2\rvert}$ is the "blobness" measure. It approaches $0$ for line-like structures (vessels) and $1$ for blob-like structures.
- $S = \sqrt{\lambda_1^2 + \lambda_2^2}$ is the "structureness" measure, which suppresses flat background regions where both eigenvalues are small.
- $\beta$ and $c$ are thresholds controlling the sensitivity of $R_B$ and $S$.

## Vessel Enhancement

Hessian-based filters are a staple in medical imaging pipelines, particularly in domains like ophthalmology for extracting the retinal vascular network from fundus images or OCT scans, for a few reasons:

1. *Shape Specificity (Tubes vs. Blobs)*: The eigenvalue analysis inherently distinguishes linear vessels from spherical pathologies (like microaneurysms or hemorrhages) and flat backgrounds. The filter mathematically isolates and enhances only the "tubular" geometry.
2. *Multi-Scale Detection*: Vascular networks contain thick main arteries and tiny capillary beds. Because the Hessian matrix is calculated using a Gaussian scale $\sigma$, the filter can be applied across a range of scales. The final output is simply the maximum vesselness response across all $\sigma$ values, allowing it to seamlessly enhance vessels of varying thicknesses simultaneously {% cite Frangi1998 --file references %}.
3. *Illumination Robustness*: Because it relies on second-order derivatives, the filter is highly robust to smooth, gradual changes in background illumination {% cite Survarachakan2021 --file references %}. This is vital for clinical imaging where uneven lighting, corneal reflections, or low contrast are common artifacts.
