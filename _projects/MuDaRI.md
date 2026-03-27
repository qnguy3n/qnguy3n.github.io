---
layout: page
title: MuDaRI
description: Multilabel Dataset of Retinal Images
img: assets/img/MuDaRI/data_aug.png
importance: 3
category: work
related_publications: true
---

In this project, we curated one of the first  Multilabel Dataset of Retinal Images for Detection of Multiple Ocular Disease.

This project was presented at he 4th [Asia Pacific Tele-Ophthalmology Society (APTOS) Symposium 2019](https://2019.asiateleophth.org), Chennai, India. [Slides](https://drive.google.com/file/d/1xU4NCehv5JpsWe5GjpTsvY2rQV8vt3yw/view)


This work also received Best Poster Presentation, Southeast Asia Machine Learning School (SEAMLS) 2019, Jakarta, Indonesia.


## Intro
The Problem
- Current retinal dataset (like EyePACS) only single label images, and mostly about DR or AMD or Glaucoma
- if we build AI on this, awkward situation where we have to say to patient something like this:
"I’m sorry, you’re not having Diabetic Retinopathy but we don’t know if you’re having other disease either.”

So, We need more datasets that
- Cover more eye diseases
- Have multiple diseases per images


## Building the dataset
### Methods
There are human intervention (check) along the pipeline.
CNN can be useful not only to classify diseases but also to process and to clean the datasets.


### Results
“Other” 
Less than 500 occurrences
Includes: Posterior Uveitis, Eye Infections, Macular Pucker, Vitreous/Retinal Hemorrhage, Posterior Capsular Opacification, Retinal Detachment/Breaks, Laser Scars, Hereditary Retinal Dystrophy, Myopia, Other Disorders On Fundus, Central Serous Chorioretinopathy, Glaucoma Suspect, Hypertensive Retinopathy, Chorioretinal Atrophy, Large Cup, Chorioretinal Neovascularization
