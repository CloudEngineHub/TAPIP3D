<div align="center">

# TAPIP3D: Tracking Any Point in Persistent 3D Geometry

<a href="https://arxiv.org/abs/XXXX.XXXXX"><img src='https://img.shields.io/badge/arXiv-Paper-red?logo=arxiv&logoColor=white' alt='arXiv'></a>
<a href='https://tapip3d.github.io'><img src='https://img.shields.io/badge/Project_Page-Website-green?logo=googlechrome&logoColor=white' alt='Project Page'></a>
<!-- <a href='https://huggingface.co/spaces/your-username/project'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Live_Demo-blue'></a> -->

</div>

<img src="/media/teaser1.gif" width="100%" alt="TAPIP3D overview">

## Overview
**TAPIP3D** is a method for long-term 3D point tracking in monocular RGB and RGB-D video sequences. It introduces a 3D feature cloud representation that lifts image features into a persistent world coordinate space, canceling out camera motion and enabling accurate trajectory estimation across frames.

## Installation
### Installing dependencies

1. Prepare the environment
```bash
conda create -n tapip3d python=3.10
conda activate tapip3d

pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 "xformers>=0.0.27" --index-url https://download.pytorch.org/whl/cu124
pip install torch-scatter -f https://data.pyg.org/whl/torch-2.4.1+cu124.html
pip install -r requirements.txt
```

2. Compile pointops2

```bash
cd third_party/pointops2
LIBRARY_PATH=$CONDA_PREFIX/lib:$LIBRARY_PATH python setup.py install
cd ../..
```

3. Compile megasam
```bash
cd third_party/megasam/base
LIBRARY_PATH=$CONDA_PREFIX/lib:$LIBRARY_PATH python setup.py install
cd ../../..
```

### Downloading checkpoints

Download our TAPIP3D model checkpoint [here](https://huggingface.co/zbww/tapip3d/resolve/main/tapip3d_final.pth) to `checkpoints/tapip3d_final.pth`

If you want to run TAPIP3D on monocular videos, you need to prepare the following checkpoints manually to run MegaSAM:

1. Download the DepthAnything V1 checkpoint from [here](https://huggingface.co/spaces/LiheYoung/Depth-Anything/resolve/main/checkpoints/depth_anything_vitl14.pth) and put it to `third_party/megasam/Depth-Anything/checkpoints/depth_anything_vitl14.pth`

2. Download the RAFT checkpoint from [here](https://drive.google.com/drive/folders/1sWDsfuZ3Up38EUQt7-JDTT1HcGHuJgvT) and put it to `third_party/megasam/cvd_opt/raft-things.pth`

Additionally, the checkpoints of [MoGe](https://wangrc.site/MoGePage/) and [UniDepth](https://github.com/lpiccinelli-eth/UniDepth.git) will be downloaded automatically when running the demo. Please make sure your network connection is available.

## Demo Usage

We provide a simple demo script `inference.py`, along with sample input data located in the `demo_inputs/` directory.

The script accepts as input either an `.mp4` video file or an `.npz` file. If providing an `.npz` file, it should follow the following format:

- `video`: array of shape (T, H, W, 3), dtype: uint8
- `depths` (optional): array of shape (T, H, W), dtype: float32
- `intrinsics` (optional): array of shape (T, 3, 3), dtype: float32
- `extrinsics` (optional): array of shape (T, 4, 4), dtype: float32

For demonstration purposes, the script uses a 32x32 grid of points at the first frame as queries.

Please first download the demo videos from [here](https://huggingface.co/zbww/tapip3d/tree/main/demo_inputs) and put them in the `demo_inputs/` directory.

### Inference with Monocular Video

By providing an video as `--input_path`, the script first runs [MegaSAM](https://github.com/mega-sam/mega-sam) with [MoGe](https://wangrc.site/MoGePage/) to estimate depth maps and camera parameters. Subsequently, the model will process these inputs within the global frame.

**Demo 1**

<img src="/media/demo1.gif" width="100%" alt="Demo 1">

To run inference:

```bash
python inference.py --input_path demo_inputs/sheep.mp4 --checkpoint checkpoints/tapip3d_final.pth --resolution_factor 2
```

An npz file will be saved to `outputs/inference/`. To visualize the results:

```bash
python visualize.py <result_npz_path>
```

**Demo 2**

<img src="/media/demo2.gif" width="100%" alt="Demo 2">

```bash
python inference.py --input_path demo_inputs/pstudio.mp4 --checkpoint checkpoints/tapip3d_final.pth --resolution_factor 2
```

**Inference with Known Depths and Camera Parameters**

If an `.npz` file containing all four keys (`rgb`, `depths`, `intrinsics`, `extrinsics`) is provided, the model will operate in an aligned global frame, generating point trajectories in world coordinates.

**Demo 3**

<img src="/media/demo3.gif" width="100%" alt="Demo 3">

```bash
python inference.py --input_path demo_inputs/dexycb.npz --checkpoint checkpoints/tapip3d_final.pth --resolution_factor 2
```

## Citation
If you find this project useful, please consider citing:

```
@misc{your2025tapip3d,
    title={TAPIP3D: Tracking Any Point in Persistent 3D Geometry},
    author={Your Name and Collaborators},
    year={2025},
    eprint={XXXX.XXXXX},
    archivePrefix={arXiv},
    primaryClass={cs.CV}
}
```