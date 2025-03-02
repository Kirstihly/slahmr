# Decoupling Human and Camera Motion from Videos in the Wild

Official PyTorch implementation of the paper Decoupling Human and Camera Motion from Videos in the Wild

[Project page](https://vye16.github.io/slahmr/) | [ArXiv](https://arxiv.org/abs/2302.12827)

<img src="./teaser.png">

##  [<img src="https://i.imgur.com/QCojoJk.png" width="40"> You can run SLAHMR in Google Colab](https://colab.research.google.com/drive/1IFvek5DSgKb80vtSvXAXh1xmBFMJuxeL?usp=sharing)

## Getting started
This code was tested on Ubuntu 22.04 LTS and requires a CUDA-capable GPU.

1. Clone repository and submodules
    ```
    git clone --recursive https://github.com/vye16/slahmr.git
    ```
    or initialize submodules if already cloned
    ```
    git submodule update --init --recursive
    ```

2. Set up conda environment. Run 
    ```
    source install.sh
    ```

    <details>
        <summary>We also include the following steps for trouble-shooting.</summary>

    * Create environment
        ```
        conda env create -f env.yaml
        conda activate slahmr
        ```
        We use PyTorch 1.13.0 with CUDA 11.7. Please modify according to your setup; we've tested successfully for PyTorch 1.11 as well.
        We've also included `env_build.yaml` to speed up installation using already-solved dependencies, though it might not be compatible with your CUDA driver.

    * Install current source repo
        ```
        pip install -e .
        ```

    * Install ViTPose
        ```
        pip install -v -e third-party/PHALP_plus/ViTPose
        ```

    * Install DROID-SLAM (will take a while)
        ```
        cd third-party/DROID-SLAM
        python setup.py install
        ```
    </details>

3. Download models from [here](https://drive.google.com/file/d/1GXAd-45GzGYNENKgQxFQ4PHrBp8wDRlW/view?usp=sharing). Run
    ```
    ./download_models.sh
    ```
    or
    ```
    gdown https://drive.google.com/uc?id=1GXAd-45GzGYNENKgQxFQ4PHrBp8wDRlW
    unzip -q slahmr_dependencies.zip
    rm slahmr_dependencies.zip
    ```

    All models and checkpoints should have been unpacked in `_DATA`.


## Fitting to an RGB video:
For a custom video, you can edit the config file: `slahmr/confs/data/video.yaml`.
Then, from the `slahmr` directory, you can run:
```
python run_opt.py data=video run_opt=True run_vis=True
```

We use hydra to launch experiments, and all parameters can be found in `slahmr/confs/config.yaml`.
If you would like to update any aspect of logging or optimization tuning, update the relevant config files.


## Fitting to specific datasets:
We provide configurations for dataset formats in `slahmr/confs/data`:
1. Posetrack in `slahmr/confs/data/posetrack.yaml`
2. Egobody in `slahmr/confs/data/egobody.yaml`
3. 3DPW in `slahmr/confs/data/3dpw.yaml`
4. Custom video in `slahmr/confs/data/video.yaml`

**Please make sure to update all paths to data in the config files.**

We include tools to both process existing datasets we evaluated on in the paper, and to process custom data and videos.
We include experiments from the paper on the Egobody, Posetrack, and 3DPW datasets.

If you want to run on a large number of videos, or if you want to select specific people tracks for optimization,
we recommend preprocesing in advance. 
For a single downloaded video, there is no need to run preprocessing in advance.

From the `slahmr/preproc` directory, run PHALP on all your sequences
```
python launch_phalp.py --type <DATASET_TYPE> --root <DATASET_ROOT> --split <DATASET_SPLIT> --gpus <GPUS>
```
and run DROID-SLAM on all your sequences
```
python launch_slam.py --type <DATASET_TYPE> --root <DATASET_ROOT> --split <DATASET_SPLIT> --gpus <GPUS>
```
You can also update the paths to datasets in `slahmr/preproc/datasets.py` for repeated use.

Then, from the `slahmr` directory,
```
python run_opt.py data=<DATA_CFG> run_opt=True run_vis=True
```

We've provided a helper script `launch.py` for launching many optimization jobs in parallel.
You can specify job-specific arguments with a job spec file, such as the example files in `job_specs`,
and batch-specific arguments shared across all jobs as
```
python launch.py --gpus 1 2 -f job_specs/pt_val_shots.txt -s data=posetrack exp_name=posetrack_val
```

We've also provided a separate `run_vis.py` script for running visualization in bulk.

## BibTeX

If you use our code in your research, please cite the following paper:
```
@inproceedings{ye2023slahmr,
    title={Decoupling Human and Camera Motion from Videos in the Wild},
    author={Ye, Vickie and Pavlakos, Georgios and Malik, Jitendra and Kanazawa, Angjoo},
    booktitle={IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
    month={June},
    year={2023}
}
