<h1 align="center">
[CoRL 2026] ReSiReg: Towards Spatially Consistent Semantics in Language-Conditioned Robotic Tasks
</h1>

<h3 align="center">
Simon Schwaiger<sup>1,2</sup>, David Seyser<sup>2</sup>, Alessandro Scherl<sup>2,3</sup>, Wilfried Wöber<sup>4</sup>, Gerald Steinbauer-Wagner<sup>1</sup>
</h3>

<p align="center">
<sup>1</sup>Graz University of Technology<br>
<sup>2</sup>University of Applied Sciences Technikum Wien<br>
<sup>2</sup>University of Alicante<br>
<sup>4</sup>University of Natural Resources and Life Sciences Vienna
</p>

<p align="center">
🌐 <a href="https://resireg.github.io">resireg.github.io</a>
</p>

<p align="center">
  <b>🟢 Try our Demo Space on Huggingface: <a href="https://huggingface.co/spaces/SimonSchwaiger/resireg-playground">ReSiReg Mini ViT-S VLM Playground</a></b>
</p>

***************************************

## 📰 News

* [04.09.2026] ReSiReg has been accepted at the Conference on Robot Learning (CoRL 2026) in Austin! 🎉 Hope to see you there!
* [22.06.2026] Huggingface release of [our ViT-S VLM based on EUPE (Link)](https://huggingface.co/collections/SimonSchwaiger/spatially-consistent-vision-language-models-resireg).
* [17.06.2026] Paper preprint release.

***************************************

## 🚀 Installation

To run everything, a Python environment with PyTorch is required; ideally it also has ROS 2 installed (ROS is not required for the eval scripts). We used the following [Docker-based workspace (with nvidia config option for ROS 2 humble)](https://github.com/SimonSchwaiger/ros-ml-container). Inference supports and auto-detects CPU, Nvidia's CUDA and Apple's MPS.

0. Install prerequisites: Docker (Nvidia Driver and Nvidia Container Toolkit for CUDA)
1. Clone ros-ml-container: `git clone https://github.com/SimonSchwaiger/ros-ml-container && cd ros-ml-container`
2. Clone this repository: `cd app && git clone https://github.com/SimonSchwaiger/resireg && cd ..`
3. Copy paper requirements into ros-ml-container for automatic dependency resolving: `cp app/resireg/paper_environment/requirements.txt .`
4. Build and run the Docker container. The first dependency install will take quite some time, builds will be cached. The command will provide a shell within the docker container: `GRAPHICS_PLATFORM=nvidia DOCKER_RUN_ARGS="--net=host --ipc=host -v $PWD/cache:/root/.cache" bash buildandrun.sh`. Inside the Docker container, this (resireg) repository will be mounted at `/app/resireg`

Checkpoints will be automatically downloaded and cached upon first run.

## Bring your own environment

It is possible to bring your own environment. For reproducibility, our exact environment is listed in `resireg/paper_environment/exact_requirements.txt`. The paper uses Ubuntu 22.04, Nvidia driver 555.58.02 with CUDA 11.8 in the container. ROS 2 version is humble. The paper runs quantitative experiments in full FP32 on CUDA; the repo supports autocasting for reduced memory and required compute (e.g., on Jetson).

We recommend pinning the main Python dependencies depending on the target hardware/setup and have the rest be resolved depending on the environment. Especially the torch and torchvision versions might have to be tweaked based on target hardware (see `resireg/paper_environment/requirements.txt` as reference). 

* torch==2.6.0
* torchvision==0.21.0
* transformers
* huggingface_hub
* datasets
* scikit-learn
* opencv-python-headless==4.11.0.86
* matplotlib==3.10.1

Depending on the task and backbone, further dependencies are required:

* **3D Mapping**
    * `open3d==0.19.0`

* **Segmentation Prior/Mask Refinement**
    * `git+https://github.com/facebookresearch/sam2.git@2b90b9f5ceec907a1c18123530e92e794ad901a4#egg=sam-2`

* **Maskclip**
    * `ftfy==6.3.1`
    * `regex==2024.11.6`

* **RADIO**
    * `timm`
    * `open_clip_torch`

* **OTAS/Dino.txt**
    * `einops`
    * `safetensors`

* **TIPSv2**
    * `sentencepiece`

***************************************

## 📄 Reproducing Paper Results

We include end-to-end eval run scripts for quantitative results. The processing pipeline is configurable using json files stored in `./src/configs`. All commands have to be executed in the 

### Dataset Download and Setup

* **ADE20K** is used from Hugging Face and will be automatically downloaded upon first use.
* **ORAD3D** validation set must be manually downloaded and placed into `resireg/eval_data/ORAD-3D`.
    Download the official release from the [ORAD-3D GitHub repository](https://github.com/chaytonmin/ORAD-3D-Dataset-For-Off-Road-AD). Extract it using their recommended `training|validation|testing` layout. The resulting path should be `eval_data/ORAD-3D/validation/<sequence>/{image_data,gt_image,gt_image_multi_seg}`. Only the validation split is required for the evaluation.
* **ScanNet3D**
    Register for access via the [official ScanNet repository](https://github.com/ScanNet/ScanNet). After approval, place the official `download-scannet.py` script at `resireg/3d_eval_data/download-scannet.py`. Then download and unpack the 12 RayFronts-style validation scenes used in Tab. 2 with `cd 3d_eval_data && bash download_val.sh && bash extract_val.sh`. Data lands in `3d_eval_data/scannet_validation`.


### Experiment 1 - 2D OVSS (Tab. 1)

Run the following commands for both (*ade20k* and *orad3d*) datasets. You can omit one of the datasets if you want to evaluate separately.

```bash
python eval_data/run_eval_matrix.py --suite exp1_1_baselines --datasets ade20k,orad3d # Baselines
python eval_data/run_eval_matrix.py --suite exp1_2_self_calib --datasets ade20k,orad3d # Self-Calibration
python eval_data/run_eval_matrix.py --suite exp1_3_resi_lite --datasets ade20k,orad3d # ReSiReg Lite
python eval_data/run_eval_matrix.py --suite exp1_4_resi_full --datasets ade20k,orad3d # ReSiReg Full
```

### Experiment 2 - 3D Feature Aggregation (Tab. 2)

```bash
python 3d_eval_data/run_eval_matrix_3d.py --suite exp2
```

### Quantitative Ablations (Tab. 3)

```bash
bash cce_ablation/run_rebuttal_ablation.sh # Tab. 3a)
bash ade_clip_layer_sweep_ablation/run_rebuttal_ablation.sh # Tab. 3b)
bash ade_seg_prior_ablation/run_rebuttal_ablation.sh # Tab. 3c)
```

***************************************

## 🎮 Inference

We use the inference helper and ROS 2 node of [OTAS (link)](https://github.com/SimonSchwaiger/otas). The demo notebook shows how to use the convenience wrappers and ROS 2 node and real-time visualiser. You can even import from and export to Nerfstudio datasets for easy reconstruction! See the notebook at [`./demo.ipynb`](./demo.ipynb).

***************************************

## 🙏 Acknowledgement

We thank all the works cited in our paper for their contributions! Without them, this research would not have been possible.

This repository adapts code from the following repositories:

* [OTAS](https://github.com/SimonSchwaiger/otas)
* [Rayfronts](https://github.com/RayFronts/RayFronts)
* [Radseg](https://github.com/RADSeg-OVSS/RADSeg)
* [SC-CLIP](https://github.com/SuleBai/SC-CLIP)
* [Maskclip_onnx](https://github.com/RogerQi/maskclip_onnx)
