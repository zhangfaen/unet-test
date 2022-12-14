# U-Net: Semantic segmentation with PyTorch
<img src="https://raw.githubusercontent.com/zhangfaen/common/master/unet-logo.png" />

![input and output for a random image in the test dataset](https://i.imgur.com/GD8FcB7.png)


Customized implementation of the [U-Net](https://arxiv.org/abs/1505.04597) in PyTorch for Kaggle's [Carvana Image Masking Challenge](https://www.kaggle.com/c/carvana-image-masking-challenge) from high definition images.

- [U-Net: Semantic segmentation with PyTorch](#u-net-semantic-segmentation-with-pytorch)
  - [Quick start](#quick-start)
    - [Without Docker](#without-docker)
  - [Description](#description)
    - [Training](#training)
    - [Prediction](#prediction)
  - [Weights & Biases](#weights--biases)
  - [Pretrained model](#pretrained-model)
  - [Data](#data)

## Quick start

### Without Docker

1. [Install CUDA](https://developer.nvidia.com/cuda-downloads)

2. [Install PyTorch](https://pytorch.org/get-started/locally/)

3. Install dependencies
```bash
pip install -r requirements.txt
```

## Description
This model was trained from scratch with 5k images and scored a [Dice coefficient](https://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient) of 0.988423 on over 100k test images.

It can be easily used for multiclass segmentation, portrait segmentation, medical segmentation, ...


### Training

```console
> python train.py -h
usage: train.py [-h] [--epochs E] [--batch-size B] [--learning-rate LR]
                [--load LOAD] [--scale SCALE] [--validation VAL] [--amp]

Train the UNet on images and target masks

optional arguments:
  -h, --help            show this help message and exit
  --epochs E, -e E      Number of epochs
  --batch-size B, -b B  Batch size
  --learning-rate LR, -l LR
                        Learning rate
  --load LOAD, -f LOAD  Load model from a .pth file
  --scale SCALE, -s SCALE
                        Downscaling factor of the images
  --validation VAL, -v VAL
                        Percent of the data that is used as validation (0-100)
  --amp                 Use mixed precision
  
For example
> nohup python train.py --epochs 10 --batch-size 5 --scale 0.3 --trained-model-file trained-models/unet-model-scale0.3-batchsize5.pth &

```
Notes:
  if cuda out of memory, consider down set, even to 1; 
  11GB GPU card (2080Ti), support batch size 5 and scale 0.3 (aka image resize to (384, 575))
  the Carvana dataset has images with resolution 1918 * 1280.

By default, the `scale` is 0.5, so if you wish to obtain better results (but use more memory), set it to 1.

Automatic mixed precision is also available with the `--amp` flag. [Mixed precision](https://arxiv.org/abs/1710.03740) allows the model to use less memory and to be faster on recent GPUs by using FP16 arithmetic. Enabling AMP is recommended.


### Prediction

After training your model and saving it to `MODEL.pth`, you can easily test the output masks on your images via the CLI.

To predict a single image and save it:

`python predict.py -i image.jpg -o output.jpg`

To predict a multiple images and show them without saving them:

`python predict.py -i image1.jpg image2.jpg --viz --no-save`

```console
> python predict.py -h
usage: predict.py [-h] [--model FILE] --input INPUT [INPUT ...] 
                  [--output INPUT [INPUT ...]] [--viz] [--no-save]
                  [--mask-threshold MASK_THRESHOLD] [--scale SCALE]

Predict masks from input images

optional arguments:
  -h, --help            show this help message and exit
  --model FILE, -m FILE
                        Specify the file in which the model is stored
  --input INPUT [INPUT ...], -i INPUT [INPUT ...]
                        Filenames of input images
  --output INPUT [INPUT ...], -o INPUT [INPUT ...]
                        Filenames of output images
  --viz, -v             Visualize the images as they are processed
  --no-save, -n         Do not save the output masks
  --mask-threshold MASK_THRESHOLD, -t MASK_THRESHOLD
                        Minimum probability value to consider a mask pixel white
  --scale SCALE, -s SCALE
                        Scale factor for the input images

For examples

> python predict.py -i ./test-images-from-internet/test1.jpg  -o ./test-images-from-internet/test1-mask-by-unet_carvana_scale0.5_epoch2-by-milesial.jpg --model trained-models/unet_carvana_scale0.5_epoch2-by-milesial.pth


> python predict.py -i ./test-images-from-internet/test1.jpg  -o ./test-images-from-internet/test1-mask-by-unet-model-scale0.3-batchsize5-by-zhangfaen.jpg --model trained-models/unet-model-scale0.3-batchsize5-by-zhangfaen.pth --bilinear

> python predict.py -i ./test-images-from-internet/test2.jpg  -o ./test-images-from-internet/test2-mask-by-unet_carvana_scale0.5_epoch2-by-milesial.jpg --model trained-models/unet_carvana_scale0.5_epoch2-by-milesial.pth

> python predict.py -i ./test-images-from-internet/test2.jpg  -o ./test-images-from-internet/test2-mask-by-unet-model-scale0.3-batchsize5-by-zhangfaen.jpg --model trained-models/unet-model-scale0.3-batchsize5-by-zhangfaen.pth --bilinear

```
You can specify which model file to use with `--model MODEL.pth`.

## Weights & Biases

The training progress can be visualized in real-time using [Weights & Biases](https://wandb.ai/).  Loss curves, validation curves, weights and gradient histograms, as well as predicted masks are logged to the platform.

When launching a training, a link will be printed in the console. Click on it to go to your dashboard. If you have an existing W&B account, you can link it
 by setting the `WANDB_API_KEY` environment variable. If not, it will create an anonymous run which is automatically deleted after 7 days.


## Pretrained model
It can be loaded from torch.hub:

```python
net = torch.hub.load('milesial/Pytorch-UNet', 'unet_carvana', pretrained=True, scale=0.5)
```
Available scales are 0.5 and 1.0.

## Data
The Carvana data is available on the [Kaggle website](https://www.kaggle.com/c/carvana-image-masking-challenge/data).
I have downloaded the data and put them under data/

The input images and target masks should be in the `data/imgs` and `data/masks` folders respectively (note that the `imgs` and `masks` folder should not contain any sub-folder or any other files, due to the greedy data-loader). For Carvana, images are RGB and masks are black and white.

You can use your own dataset as long as you make sure it is loaded properly in `utils/data_loading.py`.


---

Original paper by Olaf Ronneberger, Philipp Fischer, Thomas Brox:

[U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/abs/1505.04597)

![network architecture](https://i.imgur.com/jeDVpqF.png)

