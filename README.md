# entropy-regularized-federated-optimization
This repository is the complete reproducibility package for the paper Entropy-Regularized Federated Optimization under Non-IID Data. It provides ready-to-run code, configuration files, and analysis scripts that benchmark ERFO against nine widely used baselines—FedAvg, FedProx, SCAFFOLD, FedNova, FedDyn, FedCurv, FedAdam, FedYogi, and Ditto.
Entropy-Regularized Federated Optimization (ERFO)  
================================================

This repository contains all code, scripts, and data to reproduce the experiments from
“Khan, K. Entropy-Regularized Federated Optimization for Non-IID Distributed Data”.

Directory structure
-------------------
.
├── README.txt             ← this file  
├── requirements.txt       ← Python dependencies  
├── data/                  ← place datasets here  
│   ├── UNSW-NB15/         ← extracted from UNSW.zip  
│   └── PneumoniaMNIST/    ← extracted from PneumoniaMNIST.zip  
├── src/                   ← all training & analysis scripts  
│   ├── unsw_experiment.py       ← UNSW-NB15 federated run  
│   ├── pneumonia_experiment.py  ← PneumoniaMNIST federated run  
│   ├── ablation_study.py        ← Ablation on λ₀ initialization  
│   ├── sens_analysis.py         ← Hyperparameter sweep (η vs λ)  
│   ├── utils.py                 ← data loaders, model definitions, common funcs  
│   └── plot_results.py          ← script to regenerate all figures & tables  
├── config/                ← default YAML configs  
│   ├── unsw.yaml  
│   ├── pneumonia.yaml  
│   ├── ablation.yaml  
│   └── sweep.yaml  
  

1. Setup
--------
1.1. Create & activate Python 3.8+ virtualenv:
       python3 -m venv venv
       source venv/bin/activate

1.2. Install requirements:
       pip install -r requirements.txt

2. Prepare data
---------------
2.1. Unzip the provided datasets into `data/UNSW-NB15/` and `data/PneumoniaMNIST/`.  
2.2. Each data folder should contain train/test splits as in the original paper.

3. Running the main experiments
-------------------------------

3.1. **UNSW-NB15** (5 clients, Dirichlet-α=0.5, decayed λₜ, full participation)

    python src/unsw_experiment.py \
      --config config/unsw.yaml \
      --runs 50 \                  # number of independent seeds (default: 50)
      --seed_start 1 \             # seeds from 1 … 50
      --dirichlet_alpha 0.5 \      # controls non-IID partition
      --warmup_frac 0.05 \         # 5% IID warm-up
      --local_epochs 1 \
      --batch_size 64 \
      --learning_rate 1e-3 \
      --lambda0 1e-3 \             # initial λ₀ for ERFO-Decayed
      --participation all          # use “all” or “partial”

3.2. **PneumoniaMNIST** (5 clients, balanced split, fixed λ, partial participation)

    python src/pneumonia_experiment.py \
      --config config/pneumonia.yaml \
      --runs 50 \
      --seed_start 1 \
      --warmup_frac 0.05 \
      --local_epochs 1 \
      --batch_size 64 \
      --learning_rate 1e-3 \
      --lambda0 5e-4 \             # fixed λ for ERFO-Fixed
      --participation 3            # K=3 of N=5 clients per round

4. Ablation & Sensitivity
--------------------------

4.1. **Ablation of initial λ₀** (decayed schedule on UNSW-NB15)

    python src/ablation_study.py \
      --config config/ablation.yaml \
      --dirichlet_alpha 0.5 \
      --warmup_frac 0.05 \
      --runs 50 \
      --lambda_list 0,1e-4,1e-3,1e-2

4.2. **Hyperparameter sweep** (η vs λ on CIFAR-10)

    python src/sens_analysis.py \
      --config config/sweep.yaml \
      --dataset cifar10 \
      --grid_eta 1e-2,5e-3,1e-3,5e-4,1e-4 \
      --grid_lambda 0,1e-4,5e-4,1e-3,5e-3

5. Configuration files
----------------------

Every experiment script accepts a `--config` YAML file (in `config/`) that defines all default parameters.  You can edit these files to change settings to that of paper:
```yaml
# example: config/unsw.yaml
dataset: UNSW-NB15
dirichlet_alpha: 0.5
warmup_frac: 0.05
clients: 5
participation: all
local_epochs: 1
batch_size: 64
learning_rate: 1e-3
runs: 50
seed_start: 1
lambda0: 1e-3
lambda_schedule: decayed   # options: fixed, decayed
