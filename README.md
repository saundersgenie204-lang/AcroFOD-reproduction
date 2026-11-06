 AcroFOD Reproduction: Cross-Domain Few-Shot Object Detection

[![Paper](https://img.shields.io/badge/Paper-ECCV%202022-blue)](https://link.springer.com/chapter/10.1007/978-3-031-19815-1_39)
[![Release](https://img.shields.io/badge/Release-v1.0.0-green)](https://github.com/saundersgenie204-lang/AcroFOD-reproduction/releases/tag/v1.0.0)

This repository provides a reproduction of **AcroFOD: An Adaptive Method for Cross-Domain Few-Shot Object Detection**, published at ECCV 2022.

本项目基于原作者公开的 AcroFOD 代码，在 Cityscapes 和 Foggy Cityscapes 数据集上进行实验复现，并提供 YOLOv5x source-only Baseline 对照实验及训练权重。

- **Original repository:** https://github.com/gaoypeng/AcroFOD
- **Reproduction repository:** https://github.com/saundersgenie204-lang/AcroFOD-reproduction
- **Latest release:** https://github.com/saundersgenie204-lang/AcroFOD-reproduction/releases/tag/v1.0.0

---

1. Overview

1.1 Reproduction Tasks

本次复现主要完成以下工作：

1. 配置 AcroFOD 运行环境。
2. 准备 Cityscapes 和 Foggy Cityscapes 数据集。
3. 完成 AcroFOD 跨域目标检测训练。
4. 完成 YOLOv5x source-only Baseline 训练。
5. 整理训练结果、模型权重和复现代码。
6. 发布 GitHub Release，提供模型权重下载。

1.2 Experimental Setup

本次实验采用 Cityscapes 作为源域数据集，Foggy Cityscapes 作为目标域数据集。

| Dataset | Role |
|---|---|
| Cityscapes | Source domain |
| Foggy Cityscapes | Target domain |

The experiments cover eight object categories:

`person`, `rider`, `car`, `truck`, `bus`, `train`, `motorcycle`, `bicycle`.

---

 2. Environment

The reproduction was conducted in the following environment:

| Component | Version |
|---|---|
| Operating System | Ubuntu 18.04.6 LTS |
| Python | 3.10.9 |
| PyTorch | 2.0.0 |
| CUDA | 11.7 |
| GPU | NVIDIA GeForce RTX 3080 |
| Input Size | 640 × 640 |

Install dependencies with:

```bash
pip install -r requirements.txt
```

The project is based on the YOLOv5 3.0 codebase with the AcroFOD modifications.

---

 3. Dataset Preparation

 3.1 Dataset Download

The original project provides processed datasets in YOLO format.

Original dataset download links:

| Dataset | Description | Download |
|---|---|---|
| Sim10K | Synthetic dataset, car class | [Baidu Netdisk](https://pan.baidu.com/s/1fd1hwyGkwn-cjBL5YPCAbg?pwd=juf6) |
| KITTI | Car class | [Baidu Netdisk](https://pan.baidu.com/s/1edDtirk4IX9yFnsCGrzjDg?pwd=8brv) |
| Cityscapes_car_8_1 | 8 randomly selected Cityscapes images | [Baidu Netdisk](https://pan.baidu.com/s/1VjJn4aN5w9FdXzgIosr79Q?pwd=p69u) |
| Cityscapes_car | Cityscapes, car class | [Baidu Netdisk](https://pan.baidu.com/s/1pU7NleGc-yG_JRLFjIKcxA?pwd=4ym4) |
| Cityscapes_8cls | Cityscapes, 8 classes | [Baidu Netdisk](https://pan.baidu.com/s/1lPjaHOgoh5YCJcnP1hTzDw?pwd=rg4z) |
| Cityscapes_8cls_foggy | Foggy Cityscapes, 8 classes | [Baidu Netdisk](https://pan.baidu.com/s/1q560FQw-WSFq2_NucNNfVA?pwd=bjgr) |
| Viped | Synthetic dataset | [Baidu Netdisk](https://pan.baidu.com/s/1a1SHZ4eb2q5mSyqWY2ZQmQ?pwd=a9y7) |
| coco_person_60 | 60 randomly selected COCO person images | [Baidu Netdisk](https://pan.baidu.com/s/1VqpxNbjGjAMZvOF3HBttqw?pwd=vg1m) |
| coco_person | COCO, person class | [Baidu Netdisk](https://pan.baidu.com/s/1nwr7qVAFnXM3mK2b5Ywc9g?pwd=je89) |

You can also process raw datasets into YOLO format using the tools provided in the [original repository](https://github.com/gaoypeng/AcroFOD/tree/main/tool).

 3.2 Dataset Structure

The reproduction uses the following dataset organization:

```text
datasets/
├── cityscapes_8cls/
│   └── cityscapes_8cls/
│       └── images/
│           ├── train/
│           └── val/
└── cityscapes_8cls_foggy/
    └── cityscapes_8cls_foggy/
        └── images/
            ├── train/
            └── val/
```

The dataset paths in the YAML configuration files must be modified according to your local environment.

 3.3 Dataset Configuration

Two YAML files are used in this reproduction:

- `data/city_and_foggy8_1.yaml`: AcroFOD training configuration.
- `data/cityscapes_baseline8.yaml`: Source-only Baseline configuration.

The eight categories are:

```yaml
nc: 8
names:
  - person
  - rider
  - car
  - truck
  - bus
  - train
  - motorcycle
  - bicycle
```

---

 4. Training

 4.1 AcroFOD

AcroFOD uses both source-domain Cityscapes data and target-domain Foggy Cityscapes data during training.

Main configuration:

| Parameter | Value |
|---|---|
| Training data | Cityscapes + Foggy Cityscapes |
| Validation data | Foggy Cityscapes |
| Epochs | 300 |
| Batch size | 4 |
| Image size | 640 × 640 |
| Hyperparameters | `data/hyp_aug/m1.yaml` |
| Cross-domain Copy-Paste | Enabled (`1`) |
| k_por | 0.8 |
| Device | GPU 0 |
| Workers | 0 |

To start a new training run from scratch:

```bash
python train_MMD.py \
  --img-size 640 640 \
  --batch-size 4 \
  --epochs 300 \
  --data ./data/city_and_foggy8_1.yaml \
  --cfg ./models/yolov5x.yaml \
  --hyp ./data/hyp_aug/m1.yaml \
  --weights '' \
  --cross_domain_cp 1 \
  --k_por 0.8 \
  --device 0 \
  --workers 0 \
  --name acrofod_formal
```

**Note:** The completed reproduction run used checkpoint resumption (`resume=True`). The command above is a reference command for starting a new run from scratch, not a record of the exact historical launch command.

 4.2 YOLOv5x Source-only Baseline

The Baseline uses only source-domain Cityscapes data for training and Foggy Cityscapes data for validation.

Main configuration:

| Parameter | Value |
|---|---|
| Training data | Cityscapes |
| Validation data | Foggy Cityscapes |
| Epochs | 20 |
| Batch size | 4 |
| Image size | 640 × 640 |
| Model configuration | `models/yolov5x.yaml` |
| Hyperparameters | `data/hyp_aug/m1.yaml` |
| Device | GPU 0 |
| Workers | 0 |

Training command:

```bash
python train.py \
  --img-size 640 640 \
  --batch-size 4 \
  --epochs 20 \
  --data ./data/cityscapes_baseline8.yaml \
  --cfg ./models/yolov5x.yaml \
  --hyp ./data/hyp_aug/m1.yaml \
  --weights '' \
  --device 0 \
  --workers 0 \
  --name yolov5x_source_baseline_20e
```

---

 5. Experimental Results

The following results are extracted from the last epoch recorded in each experiment's `results.txt`.

| Method | Epochs | Precision | Recall | mAP@0.5 | mAP@0.5:0.95 |
|---|---:|---:|---:|---:|---:|
| YOLOv5x Source-only Baseline | 20 | 0.2671 | 0.07176 | 0.08065 | 0.03814 |
| AcroFOD | 300 | 0.3967 | 0.4805 | 0.4579 | 0.2661 |

 5.1 Result Interpretation

In the recorded final training epoch, AcroFOD obtains higher Precision, Recall, mAP@0.5, and mAP@0.5:0.95 than the source-only Baseline.

These results provide an initial experimental observation of cross-domain detection performance under the current reproduction setup.

**Important experimental limitation:**

The two experiments differ in both training data and training duration:

- AcroFOD uses source-domain and target-domain training data for 300 epochs.
- The Baseline uses source-domain training data only for 20 epochs.

Therefore, this comparison is not a strictly controlled experiment. The performance difference cannot be attributed solely to the AcroFOD method. Further controlled experiments with matched training settings would be required to isolate the contribution of individual method components.

**Metric source:** The values above are final-epoch training-log metrics, not independently re-evaluated best-checkpoint metrics.

---

 6. Model Weights

The trained model checkpoints are available in GitHub Release v1.0.0.

**Download:** [AcroFOD Reproduction v1.0.0](https://github.com/saundersgenie204-lang/AcroFOD-reproduction/releases/tag/v1.0.0)

| Checkpoint | Description |
|---|---|
| `AcroFOD_best.pt` | AcroFOD trained model |
| `YOLOv5x_Source_Baseline_best.pt` | YOLOv5x source-only Baseline model |

 6.1 SHA256 Verification

Use the following commands to verify downloaded checkpoints:

```bash
sha256sum AcroFOD_best.pt
sha256sum YOLOv5x_Source_Baseline_best.pt
```

Expected SHA256 values:

| Checkpoint | SHA256 |
|---|---|
| `AcroFOD_best.pt` | `da959408e4b2424aec37c790f9ef4ee74dd90b6d8649029835c96ff339ef2bb5` |
| `YOLOv5x_Source_Baseline_best.pt` | `6efaf8b4cececf7e3dce681ac1ab28d9ffbc3ab219cffd2beb567e1a2db99c27` |

---

 7. Model Evaluation

The provided `test.py` script can be used to evaluate a trained checkpoint on the validation set.

Example for AcroFOD:

```bash
python test.py \
  --weights ./release/AcroFOD_best.pt \
  --data ./data/city_and_foggy8_1.yaml \
  --img-size 640 \
  --batch-size 4 \
  --device 0 \
  --task val
```

Example for the source-only Baseline:

```bash
python test.py \
  --weights ./release/YOLOv5x_Source_Baseline_best.pt \
  --data ./data/cityscapes_baseline8.yaml \
  --img-size 640 \
  --batch-size 4 \
  --device 0 \
  --task val
```

Make sure the dataset paths in the YAML files are valid before evaluation.

---

 8. Repository Structure

```text
AcroFOD-reproduction/
├── data/
│   ├── city_and_foggy8_1.yaml
│   ├── cityscapes_baseline8.yaml
│   └── ...
├── models/
├── tool/
├── utils/
├── train.py
├── train_MMD.py
├── test.py
├── test_MMD.py
├── requirements.txt
├── README.md
└── LICENSE
```

Trained model checkpoints are distributed separately through GitHub Releases to keep the source repository lightweight.

---

 9. Limitations and Reproducibility Notes

1. This reproduction is based on the publicly available AcroFOD implementation.
2. Dataset paths need to be adapted to the local environment.
3. The reported metrics are taken from the final epoch of the training logs.
4. The AcroFOD and source-only Baseline experiments use different training data and epoch counts.
5. A strictly controlled comparison and additional independent evaluation are needed for stronger conclusions.

---

 10. Citation

If you find the original paper or implementation useful, please cite:

```bibtex
@inproceedings{gao2022acrofod,
  title={AcroFOD: An Adaptive Method for Cross-Domain Few-Shot Object Detection},
  author={Gao, Yipeng and Yang, Lingxiao and Huang, Yunmu and Xie, Song and Li, Shiyong and Zheng, Wei-Shi},
  booktitle={European Conference on Computer Vision},
  pages={673--690},
  year={2022}
}
```

---

 Acknowledgements

We thank the original authors for releasing the AcroFOD implementation and processed dataset resources.

Original implementation: https://github.com/gaoypeng/AcroFOD
