
# Fault Detection of Cyber-Physical Systems Using a Transfer Learning Method Based on Pre-Trained Transformers

[Paper (Sensors 2025)](https://doi.org/10.3390/s25134164) • Authors: Pooya Sajjadi, Fateme Dinmohammadi, Mahmood Shafiee

> Pre-trained transformer + transfer learning for CPS fault detection. Pre-train on SWaT.A1, fine-tune on SWaT.A6. Best average F1 on attack class: **93.38%** with **99.02%** accuracy under 10-fold CV. 

---

## TL;DR

- **What:** A simplified Transformer (1 encoder + 1 decoder) fine-tuned via transfer learning for CPS fault detection. Includes XAI with SHAP. 
- **Why:** Outperforms common CNN/LSTM baselines on scarce, imbalanced industrial data.
- **Data:** SWaT.A1 (source, 2015, 11 days) → SWaT.A6 (target, 2019, ~4 hours). 
- **Results:** F1 (attack) **93.38% ± 2.25**, Acc **99.02% ± 0.33** (10-fold CV on A6).

---

## Method Overview

We pre-train a compact Transformer on SWaT.A1 and fine-tune on SWaT.A6 to handle domain shift between lab and later operating conditions. SHAP explains per-sensor contributions to decisions.

**Model (paper configuration):**
- Model dim: 64 | Heads: 4 | Encoder: 1 | Decoder: 1 | Positional encoding: sinusoidal  
- Classifier: FC stack (ReLU) → logits  
- Training bits: window size 16, batch 64, LR 1e-3  
- Key libs/versions (paper): Python 3.8.10, PyTorch 2.4.1 

---

## Datasets

- **SWaT.A1** (source): 7 days normal + 4 days with 41 attacks, 449,920 records @ 1 Hz. Used for supervised pre-training. 
- **SWaT.A6** (target): ~3h normal + ~1h attacks, 13,202 points. Used for fine-tuning + 10-fold CV. 

**Notes used in the paper:**
- Keep **only features common** to both A1 and A6.  
- **Z-score** normalization computed on training split; apply to val/test.  
- Sliding window length **16** for sequence generation. 

---

## Results (as reported)

| Method                                   | Acc (%)           | F1 (attack) (%)    |
|------------------------------------------|-------------------|--------------------|
| Proposed (pre-train A1 → fine-tune A6)   | **99.02 ± 0.33**  | **93.38 ± 2.25**   |
| From scratch (no transfer)               | 97.71 ± 0.90      | 85.20 ± 5.63       |
| Freeze Transformer (cls only fine-tuned) | 97.50 ± 0.50      | 83.25 ± 4.18       |
| 1D CNN + FC                              | 97.61 ± 0.81      | 84.74 ± 4.56       |
| LSTM + FC                                | 96.95 ± 0.81      | 78.64 ± 5.67       |
| 2D CNN + FC                              | 96.74 ± 0.68      | 72.78 ± 8.13       |
| FC                                       | 96.95 ± 0.81      | 79.61 ± 6.92       |

(10-fold CV on A6; p-values vs. proposed reported in paper.) 

---

## Quick Start


### 1) Environment

- Python **3.8.10**, PyTorch **2.4.1** (as per paper). 
- Create env and install deps:
  ```bash
  conda create -n cps-tl python=3.8 -y
  conda activate cps-tl
  pip install -r requirements.txt  

### 2) Data prep

-   Get **SWaT.A1** and **SWaT.A6** (publicly available; follow their access instructions).
    
    
-   Ensure both sets are aligned to the **intersection of features**.
    
-   Split A1: 70/15/15 (train/val/test). Split A6: 85/15, then **10-fold CV** on the 85% train portion; hold out 15% for final test.
    
-   Generate sequences with **window_size=16** and **stride** you prefer; standardize with z-score using train stats.
    
    

### 3) Training

-   **Pre-train** on A1 (binary: normal vs. attack) until val plateaus; save the **best checkpoint**. Early stop ~200 epochs was used in the paper to avoid overfitting.
    
    
-   **Fine-tune** on A6 with 10-fold CV. Use Adam, **LR=1e-3**, batch **64**. Pick best val checkpoint per fold; evaluate on the held-out A6 test set.
    
    

### 4) Explainability (optional)

-   Run **SHAP** on the fine-tuned model to get per-feature impacts. In the paper, top features included **AIT202, LIT301, FIT101, FIT301, MV101**.
    
##  Citation

If you use this repository, please cite:

`@article{sajjadi2025fault,
  title={Fault Detection of Cyber-Physical Systems Using a Transfer Learning Method Based on Pre-Trained Transformers},
  author={Sajjadi, Pooya and Dinmohammadi, Fateme and Shafiee, Mahmood},
  journal={Sensors},
  volume={25},
  number={13},
  pages={4164},
  year={2025},
  publisher={MDPI},
  doi={10.3390/s25134164}
}` 

----------

##  License

This project is licensed under the **CC BY 4.0 License**.  
You are free to share and adapt the work, provided appropriate credit is given.



For questions, contact:  
📧 _pooya.sajjadi@uwl.ac.uk_

----------

## Acknowledgments

The authors acknowledge the **iTrust Centre for Research in Cyber Security** for providing the SWaT dataset.