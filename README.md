# DMT Whole-Brain Dynamics: Kuramoto Model Simulation of Psychedelic fMRI

A computational neuroscience project that applies **Kuramoto oscillatory models** to simulate the effects of **N,N-Dimethyltryptamine (DMT)** on whole-brain functional dynamics, comparing simulated neural activity against empirical resting-state fMRI data.

---

## Overview

This project implements a complete neuroimaging and computational modelling pipeline to investigate how DMT alters large-scale brain dynamics. Using empirical resting-state fMRI data from a double-blind, placebo-controlled study ([Timmermann et al., 2023](https://doi.org/10.1073/pnas.2218949120)), we construct **whole-brain network models** constrained by empirical structural connectivity and evaluate whether the **Kuramoto coupled oscillator model** can replicate key dynamic signatures of the psychedelic brain state.

The analysis spans static and dynamic functional connectivity, phase-coherence dynamics, **Leading Eigenvector Dynamics Analysis (LEIDA)** for brain state extraction, and **Functional Connectivity Dynamics (FCD)** to characterise temporal variability in connectivity patterns. Both DMT and placebo (PCB) conditions are modelled and compared against their empirical counterparts.

The full analytical pipeline — from raw data loading through to statistical comparison of brain states — is implemented in a single, reproducible Jupyter notebook.

---

## Motivation

### The Promise of Psychedelic-Assisted Therapy

Psychedelic-assisted therapy has emerged as a promising alternative to traditional long-term pharmacological interventions for psychiatric conditions including **treatment-resistant depression**, **anxiety**, **PTSD**, and **end-of-life distress**. Unlike conventional antidepressants that require chronic daily administration, psychedelic therapy typically involves only one or two dosing sessions combined with structured psychotherapeutic support. Clinical trials have demonstrated that these brief interventions can produce sustained improvements in symptom severity, with some studies reporting retention of therapeutic benefits **up to 6 months post-treatment** ([Carhart-Harris et al., 2021](https://doi.org/10.1056/NEJMoa2032994)).

This is particularly significant for **treatment-resistant** populations, where conventional pharmacological approaches have limited efficacy. Psychedelics have shown the potential to produce meaningful improvements even in patients who have not responded to multiple lines of traditional treatment.

### The Knowledge Gap

Despite growing clinical evidence, **the neural mechanisms underlying the therapeutic effects of psychedelics remain poorly understood**. Theoretical frameworks such as the **REBUS (Relaxed Beliefs Under Psychedelics)** model ([Carhart-Harris & Friston, 2019](https://doi.org/10.1124/pr.118.017160)) propose that psychedelics reduce the precision weighting of top-down priors, temporarily "flattening" the brain's predictive hierarchy and enabling greater neural entropy. However, the precise mechanisms by which serotonergic 5-HT2A receptor agonism translates into large-scale changes in brain network dynamics remain largely unknown.

### Why Computational Modelling?

Computational whole-brain models offer a principled approach to bridging this gap. By simulating neural dynamics using biologically constrained models, we can test specific hypotheses about how structural connectivity gives rise to the functional signatures observed under psychedelics. This project applies the **Kuramoto oscillatory model** — a well-established framework for studying synchronisation in coupled oscillator systems — to determine whether it can replicate the dynamic functional connectivity characteristics of the DMT brain state from structural connectivity alone.

---

## Methods & Analytical Pipeline

### Data Sources

| Data | Description | Source |
|------|-------------|--------|
| **Empirical fMRI** | Resting-state BOLD from 20 healthy volunteers (within-subjects, DMT vs. placebo), TR = 2s, 840-timepoint (28 min) recordings; the first 180 timepoints (6 min post-administration) were analysed | [Timmermann et al., 2023](https://doi.org/10.1073/pnas.2218949120) — [GitHub](https://github.com/timmer500/DMT_Imaging) |
| **Structural Connectivity** | Population-averaged DTI tractography from the Human Connectome Project (HCP) | [ENIGMA Toolbox](https://enigma-toolbox.readthedocs.io/) — [Lariviere et al., 2021](https://doi.org/10.1038/s41592-021-01186-4) |
| **Brain Parcellation** | Schaefer 2018 atlas, 100 cortical parcels mapped to 7 Yeo functional networks | [Schaefer et al., 2018](https://doi.org/10.1093/cercor/bhx179) |

**Yeo Networks (7):** Visual (VIS), Somatomotor (SMN), Dorsal Attention (DAN), Salience/Ventral Attention (SAN), Limbic (LIM), Default Mode (DMN), Frontoparietal (FPN)

### Computational Model

The **Kuramoto model** describes a system of phase-coupled oscillators where each brain region is represented as an oscillator with an intrinsic natural frequency, coupled to other regions via the empirical structural connectivity matrix:

- **Natural frequencies (omega):** Estimated from empirical BOLD signals using the Hilbert transform (instantaneous frequency) and validated with Fourier spectral analysis (0.03–0.08 Hz band)
- **Structural connectivity:** Population-averaged SC matrix from ENIGMA/HCP, providing anatomical coupling strengths
- **Signal propagation delays:** Euclidean distance matrix derived from Schaefer parcellation coordinates, informing conduction delays between regions
- **Parameter optimisation:** Coupling strength (K) and noise parameters optimised using evolutionary algorithms (neurolib Evolution)
- **Hemodynamic transform:** Oscillatory output converted to simulated BOLD signal via the **Balloon-Windkessel hemodynamic model**
- **Implementation:** [neurolib](https://github.com/neurolib-dev/neurolib) framework ([Cakan et al., 2021](https://doi.org/10.1007/s12559-021-09931-9))

### Analysis Pipeline

```
Empirical BOLD ──────────────────────────────────────┐
  │                                                   │
  ├─→ Z-score normalisation                           │
  ├─→ Static FC (Pearson correlation, Fisher Z)       │
  ├─→ Hilbert transform → instantaneous phase         │
  ├─→ Phase-coherence dFC matrices                    ├──→ Statistical comparison
  ├─→ Leading eigenvector extraction (LEIDA)          │    (Mann-Whitney U,
  ├─→ FCD matrices (eigenvector cosine similarity)    │     Wilcoxon)
  └─→ K-means clustering → FC brain states            │
                                                      │
Kuramoto Model ──→ Simulated BOLD ──→ Same pipeline ──┘
```

1. **Functional Connectivity (FC):** Pearson correlation matrices computed on Z-scored BOLD, Fisher Z-transformed for group comparison
2. **Dynamic FC (dFC):** Phase-coherence matrices computed at each timepoint via the Hilbert transform
3. **Leading Eigenvector Extraction:** Dominant eigenvector of each instantaneous dFC matrix captures the primary connectivity pattern at that moment
4. **FCD Matrices:** Time × time matrices quantifying the similarity between FC patterns across the recording, revealing temporal dynamics and state switching
5. **LEIDA Brain States:** K-means clustering of leading eigenvectors to identify recurring functional connectivity states and their condition-specific probabilities
6. **Model Validation:** FCD distributions compared between empirical and simulated data (distributional comparison was done visually; no KS test is implemented in the committed notebook)

---

## Key Findings

### FCD Replication
The Kuramoto model **successfully replicated the empirical FCD dynamics** for both DMT and placebo conditions. However, the model **exaggerated the degree of global connectivity increase** associated with DMT — the simulated DMT FCD distributions showed greater deviation from placebo baselines than observed empirically. Placebo FCD distributions from the model more closely matched the empirical data, suggesting the model captures baseline resting-state dynamics more faithfully than the perturbed psychedelic state.

### LEIDA Brain State Analysis
Clustering of empirical leading eigenvectors revealed **condition-specific functional states** with distinct characteristics:

- **Lower-order network activation state:** Significantly more probable under DMT (p(DMT) = 0.41) vs. placebo (p(PCB) = 0.18), with visual, limbic, and somatosensory activation paired with DMN/FPN decoupling
- **DMN decoupling under DMT:** Near-zero eigenvector values for DMN and FPN during DMT (vs. strong anti-correlation in placebo), indicating a breakdown of the typical resting-state hierarchical organisation — consistent with the REBUS framework
- **DMT-specific global activation state:** Identified exclusively under DMT, characterised by widespread cortical activation

### Model Limitations
While the Kuramoto simulation captured macroscopic FCD dynamics, it was **unable to fully replicate the specific resting-state brain states** identified through LEIDA clustering. The model-generated data did not yield the same pattern of recurring FC states significantly associated with the psychedelic condition, suggesting that additional biological mechanisms beyond phase-coupled oscillations may be required to capture the full complexity of the psychedelic brain state.

---

## Skills, Techniques & Packages

### Computational Neuroscience
| Technique | Description |
|-----------|-------------|
| Kuramoto coupled oscillator model | Phase-coupled network model constrained by structural connectivity |
| Balloon-Windkessel hemodynamic model | Conversion of neural oscillatory activity to BOLD signal |
| LEIDA | Leading Eigenvector Dynamics Analysis for brain state extraction |
| FCD analysis | Functional Connectivity Dynamics — temporal variability of connectivity |
| Evolutionary parameter optimisation | Automated fitting of model parameters to empirical data |
| Hilbert transform | Extraction of instantaneous phase and frequency from BOLD signals |

### Neuroimaging Analysis
| Technique | Description |
|-----------|-------------|
| Static functional connectivity | Pearson correlation-based FC with Fisher Z-transform |
| Dynamic functional connectivity | Phase-coherence dFC via Hilbert transform |
| Brain parcellation | Schaefer 2018 atlas (100 regions, 7 Yeo networks) |
| Structural connectivity | Population-averaged DTI from HCP via ENIGMA |

### Statistical Methods
| Method | Application |
|--------|-------------|
| Mann-Whitney U test | Non-parametric between-group comparisons (DMT vs. PCB) |
| Wilcoxon signed-rank test | Paired within-subject comparisons |
| Fisher Z-transform | Normalisation of correlation coefficients |
| Silhouette analysis | K-means cluster validation |
| Calinski-Harabasz & Davies-Bouldin indices | Additional clustering quality metrics |
| Permutation testing | Non-parametric significance testing |

### Python Packages
| Package | Purpose |
|---------|---------|
| `neurolib` | Kuramoto model implementation, evolutionary optimisation, signal utilities |
| `nilearn` | Neuroimaging analysis, atlas fetching, brain visualisation |
| `nibabel` | Neuroimaging file I/O (NIfTI format) |
| `enigmatoolbox` | Structural connectivity from ENIGMA/HCP datasets |
| `scipy` | Signal processing (Hilbert, bandpass filtering), statistical tests, linear algebra |
| `numpy` | Core numerical computation |
| `pandas` | Data organisation and manipulation |
| `scikit-learn` | K-means clustering, silhouette scoring, preprocessing |
| `matplotlib` | Primary visualisation library |
| `seaborn` | Statistical data visualisation |
| `xarray` | Multi-dimensional labelled array handling |
| `scikit-image` | 3D surface extraction (marching cubes) for brain visualisation |

---

## Repository Structure

```
.
├── README.md                          # This file
├── LICENSE                            # MIT License
├── requirements.txt                   # Python dependencies
├── .gitignore                         # Git ignore rules
├── AlbanMalaj_CNS_DMT.ipynb           # Main analysis notebook (full pipeline)
└── AlbanMalaj_CNS_DMT_Report.pdf      # Written research report
```

---

## Getting Started

### Prerequisites
- Python 3.8+
- Jupyter Notebook or JupyterLab
- ~4 GB RAM recommended for model fitting

### Installation

```bash
git clone https://github.com/albanana321/dmt-fmri-kuramoto-simulation.git
cd dmt-fmri-kuramoto-simulation
pip install -r requirements.txt
```

**Note:** The `enigmatoolbox` package may require separate installation. See [ENIGMA Toolbox documentation](https://enigma-toolbox.readthedocs.io/en/latest/pages/01.install/index.html) for instructions.

### Data Access

The empirical fMRI data is publicly available from the original study:

```bash
git clone https://github.com/timmer500/DMT_Imaging.git
```

Set the data path before running the notebook:
```bash
export DMT_DATA_PATH="/path/to/DMT_Imaging/fMRI"
```

Or modify the `base_directory` variable in the notebook directly.

### Running the Analysis

Open `AlbanMalaj_CNS_DMT.ipynb` in Jupyter and execute cells sequentially. The notebook is organised into the following sections:

1. **Environment Setup & Data Loading** — Package imports, data retrieval
2. **Atlas & Network Organisation** — Schaefer parcellation, Yeo network mapping
3. **BOLD Extraction & Preprocessing** — Time series extraction, normalisation
4. **Functional Connectivity** — Static FC computation and visualisation
5. **Natural Frequency Estimation** — Hilbert and Fourier-based frequency extraction
6. **Kuramoto Model Fitting** — Model initialisation, parameter optimisation (DMT & PCB)
7. **Dynamic Connectivity Analysis** — dFC, FCD, leading eigenvector extraction
8. **Statistical Comparison** — Empirical vs. simulated FCD distributions
9. **LEIDA Brain State Clustering** — K-means clustering, state probability analysis

---

## References

1. Timmermann, C., Roseman, L., Schartner, M., Millière, R., Williams, L.T.J., Erritzoe, D., Muthukumaraswamy, S., Ashton, M., Benber, A., Kaelen, M., Fielding, A., Nutt, D.J., & Carhart-Harris, R.L. (2023). Human brain effects of DMT assessed via EEG-fMRI. *Proceedings of the National Academy of Sciences*, 120(13), e2218949120. https://doi.org/10.1073/pnas.2218949120

2. Carhart-Harris, R.L. & Friston, K.J. (2019). REBUS and the anarchic brain: toward a unified model of the brain action of psychedelics. *Pharmacological Reviews*, 71(3), 316–344. https://doi.org/10.1124/pr.118.017160

3. Schaefer, A., Kong, R., Gordon, E.M., Laumann, T.O., Zuo, X.N., Holmes, A.J., Eickhoff, S.B., & Yeo, B.T.T. (2018). Local-global parcellation of the human cerebral cortex from intrinsic functional connectivity MRI. *Cerebral Cortex*, 28(9), 3095–3114. https://doi.org/10.1093/cercor/bhx179

4. Larivière, S., Paquola, C., Park, B., Royer, J., Wang, Y., Benkarim, O., de Wael, R.V., Valk, S.L., Thomopoulos, S.I., Kirschner, M., Lewis, L.B., Evans, A.C., Sisodiya, S.M., McDonald, C.R., Thompson, P.M., & Bhernhardt, B.C. (2021). The ENIGMA Toolbox: multiscale neural contextualization of multisite neuroimaging datasets. *Nature Methods*, 18, 698–700. https://doi.org/10.1038/s41592-021-01186-4

5. Cabral, J., Vidaurre, D., Marques, P., Magalhães, R., Silva Moreira, P., Miguel Soares, J., Deco, G., Sousa, N., & Kringelbach, M.L. (2017). Cognitive performance in healthy older adults relates to spontaneous switching between states of functional connectivity during rest. *Scientific Reports*, 7, 5135. https://doi.org/10.1038/s41598-017-05425-7

6. Cakan, C., Jajcay, N., & Obermayer, K. (2021). neurolib: A simulation framework for whole-brain neural mass modeling. *Cognitive Computation*, 15, 1132–1152. https://doi.org/10.1007/s12559-021-09931-9

7. Carhart-Harris, R.L., Giribaldi, B., Watts, R., Baker-Jones, M., Murphy-Beiner, A., Murphy, R., Martell, J., Blemings, A., Erritzoe, D., & Nutt, D.J. (2021). Trial of psilocybin versus escitalopram for depression. *New England Journal of Medicine*, 384(15), 1402–1411. https://doi.org/10.1056/NEJMoa2032994

**Original Analysis Scripts:** The MATLAB analysis scripts used in the original DMT imaging study by Timmermann et al. (2022) are available at [github.com/timmer500/DMT_Imaging](https://github.com/timmer500/DMT_Imaging). Some of these scripts were adapted to Python for use in this project's analytical pipeline.

---

## Acknowledgements

- **Empirical data** from Timmermann et al. (2023), Centre for Psychedelic Research, Imperial College London
- **Structural connectivity** from the [ENIGMA Toolbox](https://enigma-toolbox.readthedocs.io/) / Human Connectome Project
- **Computational framework** via [neurolib](https://github.com/neurolib-dev/neurolib) by Cakan et al. (2021)
- **LEIDA methodology** adapted from [Joana Cabral's LEiDA toolbox](https://github.com/juanitacabral/LEiDA)

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## Author

**Alban Malaj**
