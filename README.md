## yutian
### setup
- from `env.yaml`
  ```bash
  conda env create -f dipex.yaml
  conda activate dipex
  ```
- manually
  ```bash
  conda create -n dipex python=3.10
  conda activate dipex
  pip install torch==2.4.0+cu121 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
  cd Open-GroundingDino/
  pip install -r requirements.txt 
  cd models/GroundingDINO/ops
  python setup.py build install
  python test.py
  cd ../../..

  ```
### trianing on COCO2017
- DOWNLOAD [Pre-trained Model](https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha/groundingdino_swint_ogc.pth), put it to  `Open-GroundingDino/`
- DOWNLOAD [COCO2017 dataset](https://cocodataset.org/#download), put it to `DATAROOT`
- CONVERT `instance.json` to `generic.jsonl`
  ```
  python tools/coco2odvg.py --input DATAROOT/coco/annotations/instances_train2017.json --output DATAROOT/coco/annotations/generic_train2017.jsonl --generic
  ```
- CHANGE `config/datasets_caod.json`
  ```
  {
    "train": [
      {
        "root": "DATAROOT/coco/train2017",
        "anno": "DATAROOT/coco/annotations/generic_train2017.jsonl",
        "label_map": "config/coco2017_ca_label_map.json",
        "dataset_mode": "odvg"
      }
    ],
    "val": [
      {
        "root": "/DATAROOT/coco/val2017",
        "anno": "DATAROOT/coco/annotations/instances_val2017.json",
        "label_map": null,
        "dataset_mode": "coco"
      }
    ]
  }
  ```
- CHANGE `train_dipex.sh`
  ```
  python -u main.py \
          --amp \
          --output_dir logs_dipex \
          -c config/cfg_coco.py \
          --datasets config/datasets_caod.json \
          --pretrain_model_path PATH/TO/groundingdino_swint_ogc.pth \
          --options text_encoder_type=bert-base-uncased \
  ```
- train
  ```
  export HF_ENDPOINT="https://hf-mirror.com"
  bash train_dipex.sh
  ```


# DiPEx

This repository provides the official PyTorch implementation of our paper, [**DiPEx: Dispersing Prompt Expansion for Class-Agnostic Object Detection**](https://openreview.net/pdf?id=NDs9Ejz4Pe), presented at [**NeurIPS 2024**](https://nips.cc/virtual/2024/poster/95458).

<div align="center">
  <img src="misc/vis_main.png" alt="Visual Results" width="70%">
</div>

## **Overview**
Class-agnostic object detection often struggles to maintain high recall rates due to the diversity of object types and their contextual complexity. To address this challenge, we introduce Dispersing Prompt Expansion (DiPEx), a self-supervised method that leverages vision-language models. DiPEx begins by training generic parent prompts, then expands the prompt with the highest semantic uncertainty into child prompts that capture finer, non-overlapping meanings while inheriting the parent's semantics. Dispersion losses ensure high inter-class discrepancy among the child prompts while preserving consistency with the parent, and the maximum angular coverage (MAC) criterion prevents over-expansion. Experimental results demonstrate that DiPEx achieves broader, more comprehensive semantic coverage, ultimately enhancing the robustness of object detection across both in-distribution and out-of-distribution scenarios.

<div align="center">
  <img src="misc/DiPEx.png" alt="DiPEx Framework" width="70%">
</div>

---

## Table of Contents
- [Installation](#installation)
  - [Requirements](#requirements)
  - [Install Open-GroundingDino](#install-open-groundingdino)
- [Getting Started](#getting-started)
  - [Dataset Preparation](#dataset-preparation)
  - [Training & Testing](#training--testing)

---

## Installation

### Requirements
Our development and testing environment includes:
- Python 3.10+
- PyTorch 2.4.0
- CUDA 12.1

### Install Open-GroundingDino
DiPEx is built on top of [Open-GroundingDino](https://github.com/longzw1997/Open-GroundingDino).   Please follow their [installation guide](https://github.com/longzw1997/Open-GroundingDino#installation) before proceeding.

---

## Getting Started

### Dataset Preparation
This repository currently supports the **COCO** and **LVIS** datasets, with plans to add more in the near future. For detailed instructions on downloading, organizing, and preparing these datasets, refer to the [Open-GroundingDino documentation](https://github.com/longzw1997/Open-GroundingDino).

### Training & Testing

#### Training
To train DiPEx as outlined in the paper, navigate to the `Open-GroundingDino` directory and run:
```bash
bash train_dipex.sh
```

#### Testing
For class-agnostic object detection evaluation, ensure that all class labels in the ground truth annotations are set to `1` (e.g., `category_id` field in COCO). Then, to evaluate the trained DiPEx model, simply run:
```bash
bash test_dipex.sh
```

---

## Acknowledgement
We extend our gratitude to the developers of [Open-GroundingDino](https://github.com/longzw1997/Open-GroundingDino) for their outstanding codebase, on which this work is built.