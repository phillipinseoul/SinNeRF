# SinNeRF: Training Neural Radiance Fields on Complex Scenes from a Single Image
* This is a revised repository of SinNeRF based on different versions of Pytorch Lightning and some additional configurations for removing training errors.
* Original resources are at: [Paper](https://arxiv.org/abs/2204.00928), [Project Page](https://vita-group.github.io/SinNeRF/)

## Changes
* Changed Pytorch-Lightning version from 0.10.0 to 1.6.5 (latest)
* If "ImportError: cannot import name 'trunc_normal' from 'utils'" occurs, 
```
cd ~/.cache/torch/hub/facebookresearch_dino_main
```
* Then rename the file utils.py as another name (e.g. utils_v.py) and update all the import code for utils.py in that directory to the new name.

## How to run (Last updated: 2022/07/26)
### 1. Train on NeRF Synthetic Dataset
* Trained on 4 GPUs (NVIDIA RTX A5000) for ??? hours.
* Initially set `--patch_size 64 --sW 6 --sH 6`, but OOM error occurred. Then changed to `--patch_size 32 --sW 12 --sH 12` and activated `precision=16` when initializing `Trainer()` in `train.py`.
* Set `PL_TORCH_DISTRIBUTED_BACKEND=gloo` to remove errors during training.
```
PL_TORCH_DISTRIBUTED_BACKEND=gloo CUDA_VISIBLE_DEVICES="3,4,5,6" python train.py  --dataset_name blender_ray_patch_1image_rot3d  --root_dir  datasets/nerf_synthetic/lego   --N_importance 64 --img_wh 400 400 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 2e-4  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name lego_s6 --with_ref --patch_size 32 --sW 12 --sH 12 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 10 --scan 4
```

## Pipeline

![](./docs/static/media/SinNeRF.drawio.01f837d9d69b1db62c00.jpg)

## Code

### Environment

```
pip install -r requirements.txt
```

### Dataset Preparation

Please download the datasets from these links:

- NeRF synthetic: Download `nerf_synthetic.zip` from https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1
- LLFF: Download `nerf_llff_data.zip` from https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1
- DTU: Download the preprocessed DTU training data from https://drive.google.com/file/d/1eDjh-_bxKKnEuz5h-HXS7EDJn59clx6V/view

Please download the depth from here: https://drive.google.com/drive/folders/13Lc79Ox0k9Ih2o0Y9e_g_ky41Nx40eJw?usp=sharing

### Training

If you meet OOM issue, try:

1. enable `precision=16`
2. reduce the patch size `--patch_size` (or `--patch_size_x`, `--patch_size_y`) and enlarge the stride size `--sH`, `--sW`

<details>
  <summary>NeRF synthetic</summary>


- Step 1
  ```
  python train.py  --dataset_name blender_ray_patch_1image_rot3d  --root_dir  ../../dataset/nerf_synthetic/lego   --N_importance 64 --img_wh 400 400 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 2e-4  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name lego_s6 --with_ref --patch_size 64 --sW 6 --sH 6 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 10 --scan 4
  ```

- Step 2
  ```
  python train.py  --dataset_name blender_ray_patch_1image_rot3d  --root_dir  ../../dataset/nerf_synthetic/lego   --N_importance 64 --img_wh 400 400 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 5e-5  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name lego_s6_4ft --with_ref --patch_size 64 --sW 4 --sH 4 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0.01 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 0 --pt_model xxx.ckpt --nerf_only  --scan 4
  ```

</details>

<details>
  <summary>LLFF</summary>

- Step 1
  ```
  python train.py  --dataset_name llff_ray_patch_1image_proj  --root_dir  ../../dataset/nerf_llff_data/room   --N_importance 64 --img_wh 504 378 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 2e-4  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name llff_room_s4 --with_ref --patch_size_x 63 --patch_size_y 84 --sW 4 --sH 4 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 10
  ```

- Step 2
  ```
  python train.py  --dataset_name llff_ray_patch_1image_proj  --root_dir  ../../dataset/nerf_llff_data/room   --N_importance 64 --img_wh 504 378 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 5e-5  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name llff_room_s4_2ft --with_ref --patch_size_x 63 --patch_size_y 84 --sW 2 --sH 2 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0.01 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 0 --pt_model xxx.ckpt --nerf_only
  ```

</details>

<details>
  <summary>DTU</summary>

- Step 1
  ```
  python train.py  --dataset_name dtu_proj  --root_dir  ../../dataset/mvs_training/dtu   --N_importance 64 --img_wh 640 512 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 2e-4  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name dtu_scan4_s8 --with_ref --patch_size_y 70 --patch_size_x 56 --sW 8 --sH 8 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 10 --scan 4
  ```

- Step 2
  ```
  python train.py  --dataset_name dtu_proj  --root_dir  ../../dataset/mvs_training/dtu   --N_importance 64 --img_wh 640 512 --num_epochs 2000 --batch_size 1  --optimizer adam --lr 5e-5  --lr_scheduler steplr --decay_step 500 1000 --decay_gamma 0.5  --exp_name dtu_scan4_s8_4ft --with_ref --patch_size_y 70 --patch_size_x 56 --sW 4 --sH 4 --proj_weight 1 --depth_smooth_weight 0  --dis_weight 0.01 --num_gpus 4 --load_depth --depth_type nerf --model sinnerf --depth_weight 8 --vit_weight 0 --pt_model xxx.ckpt --nerf_only  --scan 4
  ```

More finetuning with smaller strides benefits reconstruction quality.

</details>


### Testing

```
python eval.py  --dataset_name llff  --root_dir /dataset/nerf_llff_data/room --N_importance 64 --img_wh 504 378 --model nerf --ckpt_path ckpts/room.ckpt --timestamp test
```

Please use `--split val` for NeRF synthetic dataset.

## Acknowledgement

Codebase based on https://github.com/kwea123/nerf_pl . Thanks for sharing!

## Citation

If you find this repo is helpful, please cite:

```

@InProceedings{Xu_2022_SinNeRF,
author = {Xu, Dejia and Jiang, Yifan and Wang, Peihao and Fan, Zhiwen and Shi, Humphrey and Wang, Zhangyang},
title = {SinNeRF: Training Neural Radiance Fields on Complex Scenes from a Single Image},
journal={arXiv preprint arXiv:2204.00928},
year={2022}
}

```
