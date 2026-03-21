---
layout: post
title: History of Medical Diagnosing AI
tags: tips gpu bugs
---

**DENDRAL** system (1969), by Bruce Buchaman and Joshua Lederberg.
- Goal: Infer molecular structure from a mass spectrometer’s information
- First expert system
- Approach:
  * Based on a set of rules
  * AND-OR tree
  * AO* search algorithm

**MYCIN** expert system (1972), by Feigenbaum, Buchaman, and Edward Shortliffe 
- Goal: Diagnose blood infections
- Novelty: Incorporate “Probabilistic Reasoning”, or “certainty factors”
- Approach:
  * Based on about 450 rules
  * Difference from DENDRAL: no theoretical model 
  * Acquired knowledge from medical textbooks and expert’s interviews
  * Simple inference engine + knowledge base

**Computer-aided Diagnosis** (1972), by Horrocks et al.
- Goal: Diagnose acute abdominal illness

**Comment:** At this point, early Bayesian system suffered from a number of problems. Thus probabilistic methods for coping with uncertainty fell out of favor in AI from the 1970s to the mid-1980s

**MUNIN** (1989), by Andersen et al.
- Goal: Diagnose neuromuscular disorders

**PATHFINDER** (1991), by Heckerman
- Goal: Pathology

**CPCS** system (1994), by Pradham et al.
- Goal: internal medicine
- Approach:
  * 448 nodes, 906 links, and 8,254 conditional probability values

**References:**
- Stuart Russell, Peter Norvig 1995. Artificial Intelligence: A Modern Approach. Upper Saddle River, New Jersey.
- Horrocks et al. (1972). Computer-aided Diagnosis: Description of an Adaptable System, and Operational Experience with 2,034 Cases. British Medical Journal, 2, 5-9
- https://expertsystem101.weebly.com/dendral.html
- https://expertsystem101.weebly.com/mycin.html

**Other resources:**
- https://www.britannica.com/technology/artificial-intelligence/Expert-systems