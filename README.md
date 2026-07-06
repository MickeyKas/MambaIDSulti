# TCM-IDS: Temporal Contrastive Mamba for Multi-Attack Intrusion Detection


A novel three-stage intrusion detection framework combining **Mamba Selective State Space Models (SSM)**, **Convolutional Block Attention (CBAM)**, and **Supervised Contrastive Learning (SupCon) with hard negative mining** for multi-class attack detection across IoT and enterprise networks.

##  Papers

This repository contains the complete implementation for the following paper:

> **TCM-IDS: Temporal Contrastive Mamba for Multi-Attack Intrusion Detection Across IoT and Enterprise Networks**  

##  Key Results

| Dataset | Macro-F1 | Accuracy | AUC-OVR | Avg Latency (CPU) | Schedulable |
|---------|----------|----------|---------|-------------------|-------------|
| RT-IoT2022   | **0.9513** ± 0.0005 | 0.9836 | 0.9972 | 3.923 ms | ✓ (U=0.392) |
| CIC-IDS2017  | **0.8945** ± 0.0010 | 0.9602 | 0.9958 | 3.588 ms | ✓ (U=0.359) |

**Key highlights:**
-  CPU-only real-time inference (no GPU required)
-  Liu-Layland schedulability at T=10ms IIoT period (U < 0.828)
-  Single architecture, no dataset-specific tuning
-  5-seed statistical validation with 95% CIs
-  Cross-domain generalisation: IoT ↔ Enterprise

##  Architecture Overview


The framework consists of:
1. **Dataset-agnostic projection layer** – maps heterogeneous feature spaces to a common embedding
2. **Mamba SSM encoder** – linear-time temporal sequence modelling with selective state spaces
3. **CBAM attention** – channel and spatial attention for discriminative feature selection
4. **SupCon with hard negative mining** – maximises inter-class embedding margins
5. **Three-stage training** – contrastive pre-training → linear probe → joint fine-tuning


##  Getting Started

### Prerequisites


Python 3.9+
PyTorch 2.0+ (CPU build recommended for edge deployment)
NumPy, Pandas, scikit-learn, Matplotlib, Seaborn

### Installation

### Dataset Setup

#### RT-IoT2022
bash
# Download from Kaggle
kaggle datasets download supplejade/rt-iot2022real-time-internet-of-things
unzip rt-iot2022real-time-internet-of-things.zip -d data/rt-iot2022/


#### CIC-IDS2017
bash
# Download from Kaggle
kaggle datasets download cicdataset/cicids2017
unzip cicids2017.zip -d data/cicids2017/

### Running the Notebook

Launch Jupyter and open `MultiAttack_TCMIDS.ipynb`:

bash
jupyter notebook MultiAttack_TCMIDS.ipynb

The notebook is self-contained and will:
1. Load and preprocess both datasets
2. Train TCM-IDS on CIC-IDS2017 (from scratch)
3. Load pre-trained RT-IoT2022 model
4. Run multi-seed evaluation
5. Generate figures and latency benchmarks

### Pre-trained Models

Pre-trained weights are available:
- `outputs_tcmids/encoder_rt_iot.pth` – RT-IoT2022 encoder
- `outputs_tcmids/classifier_rt_iot.pth` – RT-IoT2022 classifier
- `outputs_tcmids_v2/encoder_cicids2017.pth` – CIC-IDS2017 encoder
- `outputs_tcmids_v2/classifier_cicids2017.pth` – CIC-IDS2017 classifier

## Repository Structure

tcm-ids/
├── MultiAttack_TCMIDS.ipynb          # Main notebook (all experiments)
├── architecture1.png                 # Architecture diagram (horizontal)
├── architecture2.png                 # Architecture diagram (vertical)
├── trainingprotocol.png              # Three-stage training protocol
├── preprocessing.png                 # Dataset preprocessing pipeline
├── evaluation.png                    # Evaluation and deployment pipeline
├── fig_confusion_matrices.png        # Confusion matrices (generated)
├── fig_perclass_f1.png               # Per-class F1 scores (generated)
├── fig_summary_metrics.png           # Summary metrics (generated)
├── fig_latency_throughput.png        # Latency benchmarks (generated)
├── requirements.txt                  # Python dependencies
└── README.md                         # This file


##  Reproducing Results

All results are reproducible from the notebook. Key cells:

| Cell | Content |
|------|---------|
| Cell 4 | RT-IoT2022 loading and evaluation |
| Cell 5 | CIC-IDS2017 training (three stages) |
| Cell 6 | Multi-seed statistical evaluation (5 seeds) |
| Cell 7 | Figure generation |
| Cell 8 | Latency benchmark with TorchScript |

### Multi-Seed Evaluation

The model is evaluated across 5 random seeds (42, 123, 456, 789, 1010) with:
- 95% confidence intervals (Student's t-distribution)
- Shapiro-Wilk normality testing
- Macro-F1, Accuracy, AUC-OVR, Kappa, MCC

##  Key Metrics by Class

### RT-IoT2022 (5 classes, 24,601 test samples)

| Class | F1 | Support |
|-------|-----|---------|
| DoS | 0.9986 | 19,026 |
| MQTT | 0.9873 | 829 |
| Reconnaissance | 0.9711 | 1,526 |
| Normal | 0.9113 | 1,671 |
| ARP_Spoofing | 0.8880 | 1,549 |

### CIC-IDS2017 (7 classes, 88,097 test samples)

| Class | F1 | Support |
|-------|-----|---------|
| FTP_Patator | 0.9868 | 1,186 |
| DDoS | 0.9715 | 25,596 |
| Benign | 0.9636 | 29,994 |
| SSH_Patator | 0.9525 | 644 |
| DoS | 0.9506 | 29,998 |
| PortScan | 0.7994 | 391 |
| Botnet | 0.6371 | 288 |

##  Real-Time Performance

| Metric | RT-IoT2022 | CIC-IDS2017 |
|--------|------------|-------------|
| Avg Latency | 3.923 ms | 3.588 ms |
| P50 Latency | 3.891 ms | 3.573 ms |
| P99 Latency | 5.985 ms | 5.608 ms |
| Inf-only Throughput | 255 pkt/s | 279 pkt/s |
| Batch Throughput | 861 pkt/s | 862 pkt/s |
| Utilisation (U) | 0.392 | 0.359 |
| Schedulable (U<0.828) | ✓ | ✓ |

**TorchScript compilation** reduces latency by ~35-39% vs eager-mode PyTorch.

## Hyperparameters

| Parameter | Value |
|-----------|-------|
| Projection dimension | 64 |
| Mamba model dimension | 128 |
| SSM state size | 16 |
| Number of Mamba layers | 3 |
| Sliding window length | 16 |
| SupCon temperature (τ) | 0.07 |
| Hard negative count (K) | 2 × n_classes |
| Joint loss weight (λ_con) | 0.3 |
| Stage 1 learning rate | 5×10⁻⁴ |
| Stage 2 learning rate | 1×10⁻³ |
| Stage 3 learning rate | 1×10⁻⁴ |
| Batch size | 512 |
| Max epochs (S1/S2/S3) | 20/10/15 |

##  Citation

If you find this work useful, please cite:


  title={TCM-IDS: Temporal Contrastive Mamba for Multi-Attack Intrusion Detection Across IoT and Enterprise Networks},
  author={Alemayehu, Mikiyas and Ghanem, Mohamed Chahine}


## License

This project is licensed under the MIT License.
