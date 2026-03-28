---
layout: page
title: EyeNote
description: Clinical Information Extraction for Ophthalmology
img: assets/img/EyeNote/sample_output.png
importance: 1
category: work
related_publications: true
toc:
  sidebar: left
_styles: |
  .container {
    max-width: 60% !important;
  }
  .block-tip {
    background-color: #f5f5f5 !important;
    border-left-color: #aaa !important;
  }
  .block-tip p {
    color: inherit !important;
    font-size: 0.875rem !important;
  }
---

### Foreword

> This article documents the continuously evolving findings of my PhD thesis, which explores Natural Language Processing methods for clinical text in ophthalmology. It is not a finished work; the project continues to grow and accumulate new findings over time.
{: .block-tip}

### How does an ophthalmic clinical corpus look like?

The corpus used in this work consists of clinical letters extracted from the electronic health records (EHR) of Moorfields Eye Hospital NHS Trust (MEH), the largest specialist eye hospital in the UK. Letters were extracted from September 2012 through August 2024, de-identified using MedCAT {% cite Kormilitzin2021 --file references %}, and converted from RTF/HTML to plain text. Of 8,422,536 attended visits on record, 6,135,939 (72.9%) had an associated clinical letter, yielding 6,350,195 letters in total. After matching letters to their corresponding visits using a 30-day date threshold, 5,814,876 letters (91.6%) were retained for analysis.

A natural first question when working with any corpus is how lexically rich it is. A corpus with a large, rapidly growing vocabulary benefits more from domain-specific language model pre-training than one that recycles the same narrow set of terms. Heaps' law {% cite Heaps1978 --file references %} provides a principled way to measure this: it models vocabulary size $V(n)$ as a function of total token count $n$ via $V(n) = k \cdot n^\beta$, where the exponent $\beta$ captures how quickly new words appear. A higher $\beta$ means the vocabulary keeps growing; a lower $\beta$ means it plateaus early.

As shown in [**Figure 1**](#fig-heaps), MEH letters have $\beta = 0.625$, the lowest among four corpora compared (NCBI-Disease {% cite Dougan2014 --file references %}: 0.638; N2C2/MIMIC-III {% cite Henry2020 Johnson2016 --file references %}: 0.69; WikiSection: 0.715). The domain-specific vocabulary of ophthalmology saturates relatively quickly, which partly explains why pre-training a language model on MEH text alone yields diminishing returns beyond a certain scale.

<figure id="fig-heaps" style="text-align:center">
  <img src="/assets/img/EyeNote/corpus_heaps_law.png" alt="Heaps' law comparison across corpora" style="width:80%">
  <figcaption><b>Figure 1:</b> Heaps' law curves for the MEH corpus compared to NCBI-Disease, N2C2 (MIMIC-III), and WikiSection. MEH has the lowest slope ($\beta = 0.625$), indicating the most constrained lexical diversity.</figcaption>
</figure>

Documentation style varies substantially across ophthalmology subspecialties. To quantify this, letter length (token count) and the number of unique clinical findings per letter were extracted across 21 subspecialties using NLTK tokenization {% cite Bird2009 --file references %} and SemEHR {% cite Wu2018 --file references %} semantic entity extraction respectively. The results, shown in [**Figure 2**](#fig-specialty), reveal two broad clusters. Complex, investigation-heavy specialties such as Electrophysiology, Neuro-Ophthalmology, Genetics, and Ocular Oncology produce verbose, semantically dense letters (around 175-275 tokens, 12-14 clinical findings on average). High-volume procedural specialties such as Cataract and Support Services use concise, standardized notes (under 100 tokens, 4-6 findings). Some specialties break the pattern: Pediatrics letters are long but finding-sparse, reflecting the developmental history that dominates these notes, while Optometry letters are lengthy but not particularly dense in clinical findings, suggesting standardized patient instruction content.

<figure id="fig-specialty" style="text-align:center">
  <img src="/assets/img/EyeNote/wordcount_numfindings_specialty.png" alt="Letter length and findings per subspecialty" style="width:100%">
  <figcaption><b>Figure 2:</b> Distribution of (A) letter length in tokens and (B) number of unique clinical findings (CUIs) per letter, across 21 ophthalmology subspecialties. Subspecialties are ordered by mean value. Blue bars indicate means, error bars show standard deviation, and red dots mark individual outliers.</figcaption>
</figure>

### How much clinical information is locked in clinical narratives?

To answer this concretely, we focused on visual acuity (VA), arguably the single most important measurement in ophthalmology and one routinely recorded at every clinic visit. VA can be documented either in the free-text clinical letter or entered directly into the structured fields of the EHR. By running a VA extraction tool over the letter corpus and comparing results against the structured database, we can estimate what fraction of VA measurements exist only in narrative form, inaccessible to any query that reads structured data alone.

Across the full corpus, **68% of visits** had VA documented somewhere in unstructured text, and critically, **35% of visits had VA recorded exclusively in unstructured letters**, with no corresponding structured entry. As [**Figure 3**](#fig-va-specialty) shows, this varies sharply by subspecialty: Ocular Oncology operates almost entirely in unstructured text (close to 100%), while specialties such as Refractive Surgery have moved to nearly fully structured recording. Electrophysiology, Strabismus, and several others retain strong unstructured-only majorities.

<figure id="fig-va-specialty" style="text-align:center">
  <img src="/assets/img/EyeNote/unstructured_vs_structured_va.png" alt="Proportion of VA documented by mode per subspecialty" style="width:100%">
  <figcaption><b>Figure 3:</b> Proportion of clinic visits with VA documented in unstructured letters only (red), structured data only (blue), or both (green), broken down by subspecialty. Subspecialties are ordered by unstructured-only proportion.</figcaption>
</figure>

This picture changes substantially when viewed over time. [**Figure 4**](#fig-va-years) traces the same breakdown annually from 2013 to 2023. Before 2019, the vast majority of subspecialties relied almost exclusively on unstructured letters for VA. A clear inflection point emerges around 2020-2021, coinciding with EHR upgrades and, likely, the operational pressures of the COVID-19 pandemic accelerating adoption of structured data entry. By 2023, hybrid or structured-only recording dominates in high-volume specialties such as Glaucoma, Cataract, and Medical Retina. However, complex and subspecialized services including Neuro-Ophthalmology, Uveitis, and Strabismus continue to rely heavily on narrative text.

<figure id="fig-va-years" style="text-align:center">
  <img src="/assets/img/EyeNote/unstructured_structured_va_years_2013_2023.png" alt="VA documentation mode by subspecialty over 2013-2023" style="width:100%">
  <figcaption><b>Figure 4:</b> Annual breakdown of VA documentation mode per subspecialty from 2013 to 2023. Red: unstructured-only; blue: structured-only; orange: both. A hospital-wide shift from narrative-dominant to structured or hybrid recording is visible from 2020 onward.</figcaption>
</figure>

Taken together, these findings make the case for information extraction clearly: even for the most routinely measured clinical variable in ophthalmology, a third of the record exists only in free text. For rarer findings, more nuanced assessments, or historical data predating the structured data era, the fraction locked in narratives is considerably higher.

### Adapting LM to ophthalmology: does pre-training BERT still work?

At the start of my PhD (late 2021 - early 2022), the prevailing method to adapt a language model (yes, BERT was considered a Large Language Model at the time) was to pre-train BERT on a large corpus of text in the target domain using semi-supervised techniques such as Masked Language Modeling (MLM) or Next Sentence Prediction (NSP). As shown in the previous sections, the ophthalmic corpus exhibits quite unique characteristics that would suggest models could benefit from domain adaptation. To our surprise, it was not what it turned out to be.

#### EyeBERT

We pre-trained two variants of BERT on 5.8 million MEH clinical letters (approximately 700M words, 4.8 GB):

- **EyeBERT(scratch)**: trained from random weight initialization
- **EyeBERT(PBM)**: continually pre-trained from PubMedBERT

Both were evaluated against BERT {% cite Devlin2018 --file references %}, BioBERT {% cite Lee2019 --file references %}, ClinicalBERT {% cite Alsentzer2019 --file references %}, DistilBERT {% cite Sanh2019 --file references %}, and PubMedBERT {% cite Gu2020 --file references %} on two named entity recognition (NER) tasks: recognizing clinical phenotypes in eye-related PubMed case reports, and in MEH clinical letters.

#### NER Results

**Table 1** and **Table 2** report partial and strict F1 scores on the PubMed and MEH test sets respectively.

<figure id="tab-ner-pubmed">
<table style="border-collapse:collapse; width:100%">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th rowspan="2">Model</th>
      <th colspan="3" style="text-align:center">Partial</th>
      <th colspan="3" style="text-align:center">Strict</th>
    </tr>
    <tr style="border-bottom:1px solid">
      <th style="text-align:center">P</th>
      <th style="text-align:center">R</th>
      <th style="text-align:center">F1</th>
      <th style="text-align:center">P</th>
      <th style="text-align:center">R</th>
      <th style="text-align:center">F1</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>SemEHR (baseline)</td><td style="text-align:center">97.29</td><td style="text-align:center">72.50</td><td style="text-align:center">83.09</td><td style="text-align:center">97.25</td><td style="text-align:center">72.47</td><td style="text-align:center">83.05</td></tr>
    <tr><td>BERT</td><td style="text-align:center">55.11</td><td style="text-align:center">67.39</td><td style="text-align:center">60.63</td><td style="text-align:center">47.59</td><td style="text-align:center">58.21</td><td style="text-align:center">52.37</td></tr>
    <tr><td>BioBERT</td><td style="text-align:center">58.62</td><td style="text-align:center">73.33</td><td style="text-align:center">65.16</td><td style="text-align:center">51.41</td><td style="text-align:center">64.30</td><td style="text-align:center">57.14</td></tr>
    <tr><td>ClinicalBERT</td><td style="text-align:center">53.29</td><td style="text-align:center">66.58</td><td style="text-align:center">59.20</td><td style="text-align:center">44.69</td><td style="text-align:center">55.83</td><td style="text-align:center">49.64</td></tr>
    <tr><td>DistilBERT</td><td style="text-align:center">52.59</td><td style="text-align:center">64.40</td><td style="text-align:center">57.90</td><td style="text-align:center">43.42</td><td style="text-align:center">53.17</td><td style="text-align:center">47.80</td></tr>
    <tr><td>PubMedBERT</td><td style="text-align:center">80.91</td><td style="text-align:center">90.88</td><td style="text-align:center">85.61</td><td style="text-align:center">78.79</td><td style="text-align:center">88.51</td><td style="text-align:center">83.37</td></tr>
    <tr><td>EyeBERT(scratch)</td><td style="text-align:center">79.48</td><td style="text-align:center">89.01</td><td style="text-align:center">83.97</td><td style="text-align:center">77.89</td><td style="text-align:center">87.23</td><td style="text-align:center">82.30</td></tr>
    <tr style="border-bottom:2px solid; font-weight:bold"><td>EyeBERT(PBM)</td><td style="text-align:center"><b>86.58</b></td><td style="text-align:center"><b>93.31</b></td><td style="text-align:center"><b>89.82</b></td><td style="text-align:center"><b>84.84</b></td><td style="text-align:center"><b>91.44</b></td><td style="text-align:center"><b>88.02</b></td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 1:</b> NER results on PubMed eye-related case reports (partial and strict matching). Best results in bold.</figcaption>
</figure>

<figure id="tab-ner-meh">
<table style="border-collapse:collapse; width:100%">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th rowspan="2">Model</th>
      <th colspan="3" style="text-align:center">Partial</th>
      <th colspan="3" style="text-align:center">Strict</th>
    </tr>
    <tr style="border-bottom:1px solid">
      <th style="text-align:center">P</th>
      <th style="text-align:center">R</th>
      <th style="text-align:center">F1</th>
      <th style="text-align:center">P</th>
      <th style="text-align:center">R</th>
      <th style="text-align:center">F1</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>SemEHR (baseline)</td><td style="text-align:center">51.56</td><td style="text-align:center">32.57</td><td style="text-align:center">39.92</td><td style="text-align:center">44.82</td><td style="text-align:center">28.31</td><td style="text-align:center">34.70</td></tr>
    <tr><td>BERT</td><td style="text-align:center">52.42</td><td style="text-align:center">63.44</td><td style="text-align:center">57.41</td><td style="text-align:center">42.29</td><td style="text-align:center">51.19</td><td style="text-align:center">46.31</td></tr>
    <tr><td>BioBERT</td><td style="text-align:center">57.66</td><td style="text-align:center">65.60</td><td style="text-align:center">61.37</td><td style="text-align:center">47.56</td><td style="text-align:center">54.11</td><td style="text-align:center">50.62</td></tr>
    <tr><td>ClinicalBERT</td><td style="text-align:center">56.60</td><td style="text-align:center">65.83</td><td style="text-align:center">60.86</td><td style="text-align:center">45.89</td><td style="text-align:center">53.37</td><td style="text-align:center">49.35</td></tr>
    <tr><td>DistilBERT</td><td style="text-align:center">52.36</td><td style="text-align:center">63.94</td><td style="text-align:center">57.57</td><td style="text-align:center">41.06</td><td style="text-align:center">50.15</td><td style="text-align:center">45.15</td></tr>
    <tr style="font-weight:bold"><td><b>PubMedBERT</b></td><td style="text-align:center"><b>62.07</b></td><td style="text-align:center">65.09</td><td style="text-align:center"><b>63.55</b></td><td style="text-align:center"><b>54.21</b></td><td style="text-align:center">56.85</td><td style="text-align:center"><b>55.50</b></td></tr>
    <tr><td>EyeBERT(scratch)</td><td style="text-align:center">49.35</td><td style="text-align:center">59.42</td><td style="text-align:center">53.92</td><td style="text-align:center">42.52</td><td style="text-align:center">51.20</td><td style="text-align:center">46.46</td></tr>
    <tr style="border-bottom:2px solid"><td>EyeBERT(PBM)</td><td style="text-align:center">56.11</td><td style="text-align:center"><b>65.71</b></td><td style="text-align:center">60.53</td><td style="text-align:center">48.91</td><td style="text-align:center"><b>57.27</b></td><td style="text-align:center">52.76</td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 2:</b> NER results on MEH clinical letters (partial and strict matching). Best results in bold.</figcaption>
</figure>

The results tell a clear but counterintuitive story. On PubMed data, EyeBERT(PBM) is the best model overall (partial F1: 89.82%), and EyeBERT(scratch) is also competitive. But on the MEH clinical letters, the very domain EyeBERT was trained on, **PubMedBERT wins** (partial F1: 63.55%), and EyeBERT(scratch) is actually the weakest neural model (53.92%). Training from scratch on domain data did not help; if anything, it hurt.

A key part of the explanation lies in entity length. MEH clinical entities are significantly longer and more complex than those in PubMed, NCBI, or BC5CDR (mean 2.71 words vs. 1.98 in PubMed). As shown in [**Figure 5**](#fig-ner-pubmed) and [**Figure 6**](#fig-ner-meh), this matters a lot: SemEHR's recall on long PubMed entities collapses to 38% (from ~84% on short ones), and EyeBERT(scratch)'s partial F1 on long MEH entities drops to 26%. PubMedBERT, pre-trained on billions of tokens of biomedical text, handles long, complex entities far more robustly.

<figure id="fig-ner-pubmed" style="text-align:center">
  <img src="/assets/img/EyeNote/ner_pubmed_wordlen.png" alt="NER performance by entity length on PubMed" style="width:100%">
  <figcaption><b>Figure 5:</b> NER performance by entity length on the PubMed test set. (A) Short entities (≤3 words); (B) long entities (>3 words). SemEHR's recall collapses on long entities; EyeBERT(PBM) remains robust.</figcaption>
</figure>

<figure id="fig-ner-meh" style="text-align:center">
  <img src="/assets/img/EyeNote/ner_meh_wordlen.png" alt="NER performance by entity length on MEH" style="width:100%">
  <figcaption><b>Figure 6:</b> NER performance by entity length on the MEH clinical letter test set. (A) Short entities; (B) long entities. EyeBERT(scratch) degrades sharply on long entities, while PubMedBERT remains the most stable.</figcaption>
</figure>

#### Pre-training Behaviour

[**Figure 7**](#fig-pretrain-steps) traces NER performance of both EyeBERT variants across pre-training steps (10K to 400K). EyeBERT(scratch) improves steeply in the first few thousand steps as it emerges from random initialization, then plateaus around 50K steps with little further gain. EyeBERT(PBM) starts from a much stronger baseline and reaches higher performance with far less additional pre-training, confirming that initialization from a large biomedical model matters more than additional domain-specific exposure.

<figure id="fig-pretrain-steps" style="text-align:center">
  <img src="/assets/img/EyeNote/pretrain_steps.png" alt="EyeBERT NER performance across pre-training steps" style="width:85%">
  <figcaption><b>Figure 7:</b> NER performance of EyeBERT(scratch) and EyeBERT(PBM) on MEH clinical letters across pre-training steps (10K to 400K). EyeBERT(scratch) plateaus after ~50K steps; EyeBERT(PBM) achieves higher scores throughout with far less domain-specific pre-training.</figcaption>
</figure>

I suspect part of the issue is probably scale. EyeBERT was pre-trained on 700M words with a batch size of 8, compared to PubMedBERT's {% cite Gu2020 --file references %} 3.2 billion words and batch size of 8,192, and BioBERT's {% cite Lee2019 --file references %} 18 billion words. The sheer volume of biomedical text that foundation models are exposed to equips them with a richer understanding of complex morphology and clinical syntax than any single-institution corpus can provide, at least at the scale we were able to run. The low lexical diversity of the MEH corpus, as shown by its Heaps' law exponent of 0.625, compounds this: there simply is not enough new vocabulary for domain-specific pre-training to offer a decisive edge over a well-initialized foundation model.

### Automatic Extraction of Visual Acuity
Around mid-2023, several reasons led me to the decision of building an end-to-end Visual Acuity (VA) extraction pipeline. First, in discussions with clinicians at MEH about what information they most wanted extracted from clinical narratives, VA came up consistently as the top request. As shown in the previous section, despite being the single most important variable in ophthalmology, roughly a third of VA measurements cannot be found in structured data, and the further back in time one looks, the harder VA is to recover. Without an automated tool, longitudinal studies have had to rely on painstaking manual extraction.

The second motivation is a gap in how clinical NLP research is typically reported. Studies usually evaluate NER and relation extraction (RE) in isolation on benchmark datasets, but in practice the two must be chained together to produce a usable extraction pipeline. How the models interact, and how errors in one stage propagate to the other, is rarely studied.

#### Pipeline

VABERT is a two-stage system. The first stage is an NER model that identifies three entity types in clinical text: VA values (e.g., "6/6", "LogMAR 0.3", "Hand Movement"), laterality (which eye), and correction type (aided, unaided, or pinhole). The second stage is a relation extraction (RE) model that links these entities, determining which VA value belongs to which eye and which correction type. Both stages use fine-tuned BERT {% cite Devlin2018 --file references %} with a BIO tagging scheme for NER and a marker-based approach for RE. Post-processing then standardizes all extracted values, including semi-qualitative descriptors like Counting Fingers (CF), Hand Movement (HM), and No Light Perception (NLP), to the LogMAR scale. The full pipeline is shown in [**Figure 8**](#fig-va-pipeline).

<figure id="fig-va-pipeline" style="text-align:center">
  <img src="/assets/img/EyeNote/va_complete_pipeline.png" alt="End-to-end VABERT pipeline" style="width:100%">
  <figcaption><b>Figure 8:</b> End-to-end VABERT pipeline. Stage 1 (NER) identifies VA values, laterality, and correction type entities. Stage 2 (RE) links them using marker tokens. Post-processing standardizes all formats to LogMAR.</figcaption>
</figure>

#### Extraction Performance

BioBERT achieved the best NER performance with a micro-F1 of **97.05%**, more than 35 percentage points above the regex baseline (61.2%). RE is harder: the best micro-F1 is **90.80%**, with the main error being positive relations misclassified as no-relation, particularly for VA-to-correction-type pairs underrepresented in training. Applied to the full 5.8 million letter corpus, VABERT identified VA in **2,425,222 letters**, recovering around 52,000 more records than a keyword search would have found.

End-to-end accuracy was externally validated against a clinician-curated cohort of 836 letters from a genetic retinal disease study, reaching **82.78%** for right eye and **72.37%** for left eye. The gap between eyes reflects a sequential documentation bias, clinicians tend to record the right eye first, making left-eye values more susceptible to attribution errors when context is sparse.

#### What Can You Do With 2.4 Million VA Records?

Applying the pipeline to the full MEH corpus unlocks analyses that would be prohibitively slow to do manually.

**Disease-level VA profiles.** Sorting 15 of the most common diagnoses by median extracted VA reveals an expected but now automatically quantifiable clinical gradient: post-operative states (macula-on retinal detachment repair, pseudophakia) cluster at the favorable end, while vitreous hemorrhage, AMD, diabetic retinopathy, and posterior subcapsular cataract sit at the worse end. [**Figure 9**](#fig-va-disease) shows the full breakdown by disease and correction type.

<figure id="fig-va-disease" style="text-align:center">
  <img src="/assets/img/EyeNote/va_sex_ct_disease.png" alt="VA distribution by disease and correction type" style="width:100%">
  <figcaption><b>Figure 9:</b> Extracted VA distributions across the 15 most frequent diagnoses (sorted by median VA) and by correction type. Automated extraction recovers the expected clinical gradient from surgical successes to degenerative diseases.</figcaption>
</figure>

**Replicating a natural history study.** As a validation of research-grade utility, VABERT was used to replicate the VA trajectory from a published natural history study of RPGR-associated retinitis pigmentosa {% cite DeSilva2023 --file references %}. Without any manual curation, the extracted data reproduced the same age-related decline pattern reported in the original study. [**Figure 10**](#fig-va-desilva) shows the comparison. The slight divergence in patients over 40 reflects temporal noise from historical VA mentions in letters rather than true pipeline error, a known limitation of note-level extraction without explicit temporal reasoning.

<figure id="fig-va-desilva" style="text-align:center">
  <img src="/assets/img/EyeNote/va_de_silva_plot.png" alt="VABERT replication of RPGR natural history study" style="width:90%">
  <figcaption><b>Figure 10:</b> Comparison of VA trajectories for RPGR-associated retinitis pigmentosa. Left: original De Silva et al. 2023 study with manual curation. Right: VABERT-extracted data from the same cohort. The pipeline replicates the published trajectory without manual annotation.</figcaption>
</figure>

**Genotype-specific visual phenotypes at scale.** The most compelling application is the Eye2Gene project, which uses extracted VA from 26,049 measurements across 2,570 patients with inherited retinal disease (IRD), spanning 36 disease genes. The heatmap in [**Figure 11**](#fig-va-e2g) reveals three biologically distinct phenotypic clusters that emerge cleanly from the extracted data alone.

- **Vision-preserving genotypes** (CHM, EFEMP1, RHO): distributions concentrated at LogMAR ≤ 0.3, indicating retained central vision despite peripheral degeneration.
- **Visual ceiling genotypes** (CNGA3, CNGB3, KCNV2, cone dystrophies and achromatopsia): tight, kurtotic distributions around LogMAR 0.7-1.0, reflecting a stationary cone dysfunction phenotype where patients have stable but permanently limited acuity.
- **Severe progression genotypes** (BBS1, RP2, RDH12): right-skewed distributions extending toward HM and LP, with discrete vertical bands at LogMAR 2.28 and 2.7 marking the encoded semi-qualitative descriptors, indicating frequent progression beyond measurable Snellen range.

These clusters align precisely with known clinical genetics, validating that the pipeline recovers biologically meaningful signal from free text at a scale that would take years to collect manually.

<figure id="fig-va-e2g" style="text-align:center">
  <img src="/assets/img/EyeNote/va_e2g_results.png" alt="Gene-specific VA distributions across 36 IRD genes" style="width:100%">
  <figcaption><b>Figure 11:</b> Heatmap of best-corrected VA distributions for 36 inherited retinal disease genes (Eye2Gene project), extracted from 26,049 VA measurements across 2,570 patients. Genes are sorted by median VA (top: worst, bottom: best). Three phenotypic clusters are visible: vision-preserving, visual ceiling, and severe progression genotypes. Vertical bands at LogMAR 1.98, 2.28, and 2.7 correspond to CF, HM, and LP encodings.</figcaption>
</figure>

The practical upshot is straightforward: a pipeline that achieves ~83% end-to-end accuracy on a held-out cohort, when applied at the scale of millions of letters, produces a phenotypic database rich enough to replicate published natural history studies and recover genotype-specific disease signatures, all without a single manual annotation beyond the initial model training.

### Federated Learning or LLM?

In late 2024, I had the opportunity to collaborate with Dr. Sophia Wang at the Byers Eye Institute, where I gained access to a VA dataset from Stanford, similar in spirit to the MEH corpus but distinctly different in letter length, documentation style, clinical complexity, and patient population. Access to two such distinct datasets from separate clinical settings on separate continents is rare, and since Dr. Sophia's lab had already done prior work on VA extraction {% cite Bernstein2024 --file references %}, this felt like a natural moment to ask a harder question: can we find a single model that generalises well across both sites?

The key constraint is that the two datasets cannot be pooled in one place due to data privacy requirements. Two approaches present themselves: federated learning {% cite McMahan2017 --file references %}, where a BERT model is trained jointly using data from both sites without ever centralising it, or a large language model pre-trained on a massive and diverse corpus, which may generalise across clinical settings without any site-specific adaptation. The full study has been published.

#### Datasets

The Stanford corpus consists of 319,756 clinical notes from ~90,000 patients, with VA annotations generated in a weakly supervised manner from semi-structured EHR fields. The MEH corpus is far smaller, 254 training and 98 test notes, with all VA entities manually annotated. The two corpora look nothing alike: Stanford notes are produced using EHR-formatting templates and follow a machine-readable, highly consistent layout, while MEH notes are written entirely by clinicians in natural free text, with implicit conventions that require clinical knowledge to parse correctly (for example, the first VA appearing in the note is assumed to be the right eye). [**Figure 12**](#fig-fl-data) illustrates this contrast.

<figure id="fig-fl-data" style="text-align:center">
  <img src="/assets/img/EyeNote/va_fl_data_example.png" alt="VA note examples from Stanford and MEH" style="width:100%">
  <figcaption><b>Figure 12:</b> Examples of VA entities from (A) Stanford Byers Eye Institute and (B) Moorfields Eye Hospital. Stanford notes, generated using EHR templates, follow a consistent structured format. MEH notes, written by clinicians, are natural free text with implicit clinical conventions.</figcaption>
</figure>

#### Results

[**Table 3**](#tab-fl-results) summarizes the micro-averaged strict F1 across all models and evaluation settings; [**Figure 13**](#fig-fl-f1) traces the F1 trajectory across 500 federated communication rounds.

<figure id="tab-fl-results">
<table style="border-collapse:collapse; width:80%; margin-left:auto; margin-right:auto">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th>Model</th>
      <th style="text-align:center">Stanford<br>F1<sub>strict</sub></th>
      <th style="text-align:center">MEH<br>F1<sub>strict</sub></th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Baseline (home site)</td><td style="text-align:center"><b>0.943</b></td><td style="text-align:center">0.427</td></tr>
    <tr><td>Cross-evaluation (other site)</td><td style="text-align:center">0.204</td><td style="text-align:center">0.502</td></tr>
    <tr><td>FedAvg</td><td style="text-align:center">0.942</td><td style="text-align:center">0.544</td></tr>
    <tr><td>STWT</td><td style="text-align:center">0.940</td><td style="text-align:center">0.519</td></tr>
    <tr><td>LLaMA-3-70B {% cite Dubey2024 --file references %}</td><td style="text-align:center">0.375</td><td style="text-align:center"><b>0.660</b></td></tr>
    <tr style="border-bottom:2px solid"><td>Mixtral-8x7B {% cite Jiang2024 --file references %}</td><td style="text-align:center">0.288</td><td style="text-align:center">0.423</td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 3:</b> Micro-averaged strict F1 scores for VA NER across models and evaluation settings. Baseline (home site): model trained and tested on the same institution. Cross-evaluation: model trained at one site, tested at the other.</figcaption>
</figure>

**The generalization gap is real and severe.** Cross-site evaluation reveals a dramatic performance collapse: the Stanford-trained model dropped from F1_strict=0.943 at home to 0.502 on MEH. The reverse was even worse, reaching only 0.204 on Stanford, because MEH's implicit free-text conventions are largely illegible to a model trained exclusively on templated notes.

**Federated learning bridges the gap without touching the data.** Both FL algorithms substantially recovered this loss. FedAvg reached F1_strict=0.544 on MEH while maintaining 0.942 on Stanford. STWT achieved comparable strict scores (0.519 MEH, 0.940 Stanford) and outperformed FedAvg on partial matching (0.844 vs. 0.814 on MEH), with markedly more stable convergence throughout training, as seen in [**Figure 13**](#fig-fl-f1). The noisy F1 curve of FedAvg across rounds contrasts sharply with STWT's smooth progression, a meaningful practical advantage when you cannot afford to run for hundreds of rounds to find the best checkpoint.

**LLMs are note-structure dependent.** LLaMA-3-70B {% cite Dubey2024 --file references %} excelled on MEH's natural free text (F1_strict=0.660), easily outperforming all BERT-based models with just five in-context examples. But it collapsed on Stanford's templated notes (F1_strict=0.375). Mixtral-8x7B {% cite Jiang2024 --file references %} fared similarly. This reveals a nuanced point: LLMs handle linguistic variation and implicit clinical conventions well, but the rigid machine-generated patterns in templated notes are precisely what discriminative BERT models learn to exploit. Neither paradigm dominates both environments.

<figure id="fig-fl-f1" style="text-align:center">
  <img src="/assets/img/EyeNote/va_fl_results_f1.png" alt="F1 curves across federated communication rounds" style="width:100%">
  <figcaption><b>Figure 13:</b> Micro-F1 scores over 500 communication rounds for Stanford (left) and MEH (right). Top row: strict evaluation; bottom row: partial evaluation. STWT (blue) converges smoothly and consistently outperforms FedAvg (orange) on MEH. The dashed purple line marks LLaMA-3-70B performance for reference.</figcaption>
</figure>

The takeaway is clear: when clinical note styles differ substantially across institutions, no single model generalises for free. FL, particularly STWT, offers a principled path to cross-institutional performance without centralising sensitive data, and is most valuable precisely in the settings where naive cross-evaluation fails the hardest.

### Toward scalable and general-purpose ophthalmic information extraction

The previous chapters demonstrated that fine-tuned BERT models can achieve strong performance on specific, well-defined extraction tasks, but each one required a custom annotated dataset. This is a bottleneck. In practice, clinical teams generate dozens of different extraction requests, each with its own schema, subspecialty, and note style. Annotating a training set for every new task is not feasible. Generative large language models offer a different path: define the task in a prompt, and the model extracts the information directly, with no fine-tuning required. The question is whether this actually works in ophthalmology, across tasks of varying complexity and clinical domains.

To answer this, I assembled the **Ophthalmic Language Understanding Benchmark (OLUB)**, a heterogeneous corpus of ten real-world datasets collected through collaborations with clinicians at MEH and the Byers Eye Institute between January 2024 and September 2025. Each dataset originated from a genuine clinical audit or research project, covering subspecialties including glaucoma, strabismus, cataract, cornea, genetics, and uveitis. The tasks span two types: **patient identification** (given a patient's aggregated notes, determine whether they meet a clinical criterion) and **clinical variable extraction** (extract structured data such as VA measurements, laterality, or treatment outcomes from individual notes). Document lengths vary by more than two orders of magnitude, from short single-visit notes (median ~130 tokens) to longitudinally aggregated patient records exceeding 88,000 words.

#### Models and setup

Seven models were evaluated across a parameter range from 8B to 70B:

- **Generalist models**: LLaMA-3.1-8B {% cite Dubey2024 --file references %}, Gemma-2-9B/27B {% cite GemmaTeam2024 --file references %}, GPT-OSS-20B {% cite OpenAI2025 --file references %}, Mixtral-8x7B {% cite Jiang2024 --file references %}, LLaMA-3.3-70B {% cite Dubey2024 --file references %}
- **Ophthalmology-specialized**: LEME-DPO 70B {% cite Gilson2024 --file references %}, trained on ~211K ophthalmology-specific instruction samples and aligned using Direct Preference Optimization

All models ran locally on hospital-grade hardware (8×NVIDIA P100 at MEH; 2×A100 at Stanford) using 4-bit quantization. Prompts followed a standardized three-part structure: task definition, optional notation for clinical abbreviations, and a strict JSON output specification enforced by an automatic retry mechanism (up to 5 attempts per sample).

#### Results

[**Table 4**](#tab-llm-perf) summarizes performance across all models and datasets.

<figure id="tab-llm-perf">
<table style="border-collapse:collapse; width:100%">
  <thead>
    <tr style="border-top:2px solid; border-bottom:1px solid">
      <th>#</th>
      <th>Dataset</th>
      <th style="text-align:center">8B</th>
      <th style="text-align:center">Gemma<br>9B</th>
      <th style="text-align:center"><b>GPT-OSS<br>20B</b></th>
      <th style="text-align:center">Gemma<br>27B</th>
      <th style="text-align:center">Mixtral<br>8x7B</th>
      <th style="text-align:center">LLaMA<br>70B</th>
      <th style="text-align:center">LEME<br>70B</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>1</td><td>GDD-Botox</td><td style="text-align:center">0.061</td><td style="text-align:center">0.780</td><td style="text-align:center"><b>0.922</b></td><td style="text-align:center">0.720</td><td style="text-align:center">0.711</td><td style="text-align:center">0.909</td><td style="text-align:center">0.770</td></tr>
    <tr><td>2</td><td>IR-Botox</td><td style="text-align:center">0.022</td><td style="text-align:center">0.680</td><td style="text-align:center"><b>0.912</b></td><td style="text-align:center">0.696</td><td style="text-align:center">0.212</td><td style="text-align:center">0.622</td><td style="text-align:center">0.338</td></tr>
    <tr><td>3</td><td>Cong. Esotropia</td><td style="text-align:center">0.984</td><td style="text-align:center">0.423</td><td style="text-align:center"><b>0.989</b></td><td style="text-align:center">0.027</td><td style="text-align:center">0.907</td><td style="text-align:center">0.986</td><td style="text-align:center">0.995</td></tr>
    <tr><td>4</td><td>YAG Cap</td><td style="text-align:center">0.711</td><td style="text-align:center">0.678</td><td style="text-align:center">0.829</td><td style="text-align:center">0.815</td><td style="text-align:center">0.738</td><td style="text-align:center"><b>0.868</b></td><td style="text-align:center">0.843</td></tr>
    <tr><td>5</td><td>RP-CMO</td><td style="text-align:center">0.630</td><td style="text-align:center"><b>0.770</b></td><td style="text-align:center">0.660</td><td style="text-align:center">0.650</td><td style="text-align:center">0.710</td><td style="text-align:center">0.630</td><td style="text-align:center">0.640</td></tr>
    <tr><td>6</td><td>PED-Postop</td><td style="text-align:center">0.721</td><td style="text-align:center">0.860</td><td style="text-align:center">0.870</td><td style="text-align:center">0.873</td><td style="text-align:center">0.851</td><td style="text-align:center"><b>0.893</b></td><td style="text-align:center">0.845</td></tr>
    <tr><td>7</td><td>VA-RPGR</td><td style="text-align:center">0.900</td><td style="text-align:center">0.951</td><td style="text-align:center">0.968</td><td style="text-align:center">0.939</td><td style="text-align:center">0.924</td><td style="text-align:center"><b>0.970</b></td><td style="text-align:center">0.950</td></tr>
    <tr><td>8</td><td>FECD-SRGR</td><td style="text-align:center">0.810</td><td style="text-align:center">0.815</td><td style="text-align:center">0.807</td><td style="text-align:center">0.818</td><td style="text-align:center">0.282</td><td style="text-align:center"><b>0.824</b></td><td style="text-align:center">0.750</td></tr>
    <tr><td>9</td><td>Byers-VA</td><td style="text-align:center">0.634</td><td style="text-align:center">0.537</td><td style="text-align:center"><b>0.867</b></td><td style="text-align:center">0.655</td><td style="text-align:center">0.678</td><td style="text-align:center">0.757</td><td style="text-align:center">0.711</td></tr>
    <tr><td>10</td><td>Byers-CAAMPUS</td><td style="text-align:center">0.449</td><td style="text-align:center">0.412</td><td style="text-align:center"><b>0.626</b></td><td style="text-align:center">0.559</td><td style="text-align:center">0.535</td><td style="text-align:center">0.596</td><td style="text-align:center">0.451</td></tr>
    <tr style="border-top:1px solid; border-bottom:2px solid; font-weight:bold"><td></td><td>Mean</td><td style="text-align:center">0.592</td><td style="text-align:center">0.691</td><td style="text-align:center"><b>0.845</b></td><td style="text-align:center">0.675</td><td style="text-align:center">0.655</td><td style="text-align:center">0.806</td><td style="text-align:center">0.729</td></tr>
  </tbody>
</table>
<figcaption style="text-align:center; margin-top:0.5em"><b>Table 4:</b> LLM performance across the 10 OLUB datasets. Best score per row in bold. 8B = LLaMA-3.1-8B; Gemma 9B/27B; GPT-OSS 20B; Mixtral 8x7B; LLaMA 70B = LLaMA-3.3-70B; LEME 70B = LEME-DPO.</figcaption>
</figure>

**Parameter count alone does not determine performance.** GPT-OSS-20B achieved the highest mean score (0.845) despite being less than a third of the size of the 70B models. It ranked first on 6 of 10 tasks and was the only model to break 0.9 on three different datasets. LLaMA-3.3-70B came second (0.806), while LEME-DPO, the ophthalmology-specialized 70B model, scored only 0.729, below even Gemma-2-9B (0.691). Alignment quality and instruction-following ability, not domain-specific pretraining, appear to be the dominant factors for structured extraction tasks.

**Task difficulty matters more than model size.** Simple patient identification tasks with short, well-structured notes (GDD-Botox, IR-Botox, Congenital Esotropia) were easy for capable models and near-impossible for small or poorly aligned ones. Long aggregated documents (Byers-CAAMPUS: median 8,282 tokens, max 88,858) degraded all models, including GPT-OSS-20B whose best score there was only 0.626. This ceiling is a fundamental challenge for LLMs on longitudinal clinical records.

#### Performance vs. efficiency

[**Figure 14**](#fig-llm-time) shows the clear performance-efficiency trade-off. Smaller models are fast but mediocre. The largest models improve accuracy but at rapidly increasing latency, with LLaMA-3.3-70B averaging 17.03 seconds per data point compared to 4.14 seconds for GPT-OSS-20B, for a gain of only 0.039 in mean score. For any pipeline processing thousands of clinical letters, this cost is rarely justifiable.

<figure id="fig-llm-time" style="text-align:center">
  <img src="/assets/img/EyeNote/llm/results_model_performance_vs_time.png" alt="Performance vs processing time for LLMs" style="width:75%">
  <figcaption><b>Figure 14:</b> Mean evaluation score versus mean adjusted processing time per data point for each model. GPT-OSS-20B occupies the favorable middle ground: highest performance with moderate latency. The largest models show diminishing returns.</figcaption>
</figure>

#### Reliability of structured output

Beyond accuracy and speed, a critical practical concern is whether models consistently produce parseable JSON, a prerequisite for automated downstream processing. As shown in [**Figure 15**](#fig-llm-reliability), first-attempt success rates vary widely and do not correlate with model size. GPT-OSS-20B achieved 99.17% valid JSON on the first attempt; Gemma-2-27B reached 99.04%. In contrast, LEME-DPO produced valid JSON on the first attempt only 76.66% of the time, the worst of all models, despite being domain-specialized. Mixtral-8x7B (83.87%) and LLaMA-3.1-8B (87.55%) also showed high failure rates. Domain-specific fine-tuning appears to trade structural discipline for semantic richness, a trade-off that is expensive in any automated pipeline.

<figure id="fig-llm-reliability" style="text-align:center">
  <img src="/assets/img/EyeNote/llm/results_attempts_distribution.png" alt="Distribution of output parsing attempts by model and project" style="width:100%">
  <figcaption><b>Figure 15:</b> Percentage distribution of output-parsing attempts (1 to 5) required to produce valid JSON, stratified by project and model. GPT-OSS-20B and Gemma-2-27B are near-ceiling; LEME-DPO and Mixtral show the most multi-attempt failures.</figcaption>
</figure>

The overall picture from this work is encouraging but realistic. LLMs, particularly well-aligned intermediate-scale models like GPT-OSS-20B, can meaningfully support real clinical extraction workflows in ophthalmology without any task-specific annotation, making them genuinely useful for new, low-resource extraction tasks. However, long aggregated documents remain a hard problem, reliability varies substantially across models in ways that are not predictable from scale, and inference latency at the 70B tier remains a bottleneck for large-scale deployment. The most viable path forward is probably a hybrid one: use LLMs to generate training data or handle rare tasks, and reserve encoder-based models for high-throughput, well-defined extractions where fine-tuning is feasible.

### Conclusion

Looking back across these chapters, a few threads run through everything.

The first is about data. A third of all visual acuity measurements at the largest specialist eye hospital in the UK exist only in free text, inaccessible to any structured query. For rarer variables, older records, or subspecialties that have not yet adopted templated data entry, the fraction locked in narrative is higher still. The premise of this work is simple: clinical NLP can unlock that data at scale, and the payoff, in terms of retrospective research, cohort discovery, and phenotypic characterization, is large enough to justify the effort.

The second thread is about what kind of model to use, and when. BERT-based models, when fine-tuned on even modest amounts of annotated data, are highly competitive and offer speed, reliability, and interpretability that matter in deployment. Pre-training on domain-specific text (EyeBERT) did not consistently outperform strong foundation models like PubMedBERT, suggesting that the volume and diversity of pre-training data matter more than domain overlap alone. For the VA extraction task specifically, a fine-tuned BioBERT model achieved 97% NER micro-F1 and recovered 2.4 million VA records from 5.8 million clinical letters, enabling analyses that would have taken years to replicate manually. For generalization across institutions with different documentation styles, federated learning, particularly STWT, proved a practical and privacy-preserving path, recovering most of the performance lost in naive cross-site transfer without pooling a single record.

The third thread is about the emerging role of LLMs. For new extraction tasks where annotation is expensive, generative LLMs with prompt-based interfaces offer genuine value: no retraining, no annotated dataset, and reasonable accuracy across diverse clinical tasks. The OLUB results show that well-aligned models at the 20B scale can match or exceed specialized 70B models on structured extraction, and that the bottleneck is usually task formulation and document length, not model size. At the same time, the failure modes, unreliable JSON output, degradation on long aggregated records, high inference cost, mean that LLMs are better positioned as a complement to rather than a replacement for encoder-based extraction at scale.

The broader ambition behind all of this is a clinical NLP infrastructure for ophthalmology that can be deployed routinely: pulling structured data from letters as they are written, enriching electronic health records retrospectively, and making large-scale phenotypic research from routine care data a realistic proposition rather than a heroic manual effort. Each chapter here is a step in that direction.

