![localizer](https://github.com/ivan-alles/localizer/workflows/CI/badge.svg)

# Localizer

[![Video Intro](/assets/youtube_thumbnail.jpg)](https://youtu.be/M1_5VaDYxK4 "Video Intro")

Localizer is a python library for 2D object detection. Popular algorithms like YOLO find axis-aligned
or rotated bounding boxes, which is only a rough description of the object location. 
In contrast, Localizer predicts accurate object coordinates and orientation. This data is essential, 
for example, in robotic applications, to plan precise motions and avoid collisions.

Check out the online [hand detector app](https://ivan-alles.github.io/localizer/) to see how to find the position and 
orientation of hands on a live camera video in a pure JavaScript code.

Localizer is a deep neural network. Provided enough training data, it:
* Reaches accuracy better than 2 pixels for position, 2 degrees for orientation, and 99.9% for classification.
* Detects 100+ object categories.
* Works with rigid and flexible object shapes and structures.
* Adapts to variations in size, point of view, and lighting.


The object detection runs in real-time on various platforms (CPU, GPU, FPGA, smartphones).

Transfer learning reduces the amount of training data and time by 10-20 times.

Localizer powered various industrial and hobby projects in:
* Robot control
* Manufacturing quality assurance
* Object counting
* Board games

## How it works

First, you create a dataset with labeled object positions and rotation angles. You also make a configuration file with
training parameters. 

Then you run the training process, which will create and train an object detection Tensorflow model. The training data 
will be generated from the dataset using data augmentation like image and color transformations. The training will
periodically compute object detection metrics to evaluate the correctness of the model. 

Then you use this model for inference to calculate 3DOF object pose estimation, category, and confidence.

For an easy start, this repository contains:
* The source code
* Examples of datasets and models
* A pretrained transfer learning model
* A hands-on demo app for training and running models

## Setup
### Windows 

1. To run neural networks on a GPU (highly recommended), 
   install the required **[prerequisites](https://www.tensorflow.org/install/gpu)** for TensorFlow 2.
2. Get the source code into your working folder.
3. Install the dependencies: `pipenv sync`.
4. Activate the pipenv environment: `pipenv shell`.
5. Add localizer to python: `set PYTHONPATH=.`.  
### JetPack 5.0.2
```shell
# clone git repo
mkdir ~/lihang
cd ~/lihang
git clone https://github.com/inlihang/localizer.git
cd localizer

# install virtualenv (Tensorflow on JetPack 5.0.2 only supports python 3.8)
sudo apt install python3.8-venv
python3.8 -m venv .venv3.8
source .venv3.8/bin/activate

# install pre-requisites
pip3 install -U numpy grpcio absl-py py-cpuinfo psutil portpicker six mock requests gast h5py astor termcolor protobuf keras-applications keras-preprocessing wrapt google-pasta setuptools testresources

# install tensorflow for python 3.8 on arm and linux
pip install https://developer.download.nvidia.com/compute/redist/jp/v502/tensorflow/tensorflow-2.10.0+nv22.10-cp38-cp38-linux_aarch64.whl

# verify the installtion (should see GPU info if successful installation)
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

pip install -r requirements.txt
export PYTHONPATH=":./.venv3.8v2/lib/python3.8/site-packages"
```

## Hands-on python demo

<img src="./assets/hands_on.gif">

To see how to detect the position and orientation of objects without writing any code, use the hands-on demo app. 
You can interactively train and run a model on images from your web camera. Run 
`python localizer/hands_on_demo.py [CAMERA_ID]` and follow the on-screen instructions. 
You can select a camera with the optional `CAMERA_ID`parameter. It is an integer with the default value of 0. 

## Converting an existing dataset

A dataset for the Localizer is a JSON list of images, each containing a list of objects 
with the position, orientation, and category:

```json
[
 {
  "image": "myimage.png",
  "objects": [
   {
    "category": 0,
    "origin": {
     "x": 100,
     "y": 200,
     "angle": 3.14159
    }
   }
  ]
 }
]
```

If you have your dataset, you need to convert it to this format.

## Creating a new dataset 

You can use [Anno](https://github.com/urobots-io/anno/) to label object detection data manually. To do this:

1. Download and install [Anno](https://github.com/urobots-io/anno/).
2. Copy `localizer\dataset_template.anno` into the directory with your images and rename it (e.g. `mydataset.anno`).
3. Open `mydataset.anno` with Anno.
4. Label objects with the **object** marker. 
5. Label images without any objects with the **empty** marker.

You can specify this anno file in the model configuration:

```json
{
  "dataset": "path/mydataset.anno"
} 
```

## Training a model
Run `python localizer\train.py PATH_TO_MODEL_CONFIG`. 

For example: `python localizer\train.py models\tools\config.json`.

This command will train and save a Tensorflow model under `models\tools\model.tf`.

## Making predictions
You can start off with the included program for prediction on images in a folder:
 
`python localizer\predict_for_images.py PATH_TO_MODEL_CONFIG IMAGES_DIR`.
 
For example: `python localizer/predict_for_images.py models/tools/config.json datasets/tools`.

Refer to the source code for details on how to find the position and orientation of an object in the prediction.

Localizer does not predict bounding boxes. If you need them and your objects have fixed sizes, you can generate 
bounding boxes using the predicted object position and orientation angle.

## Transfer learning
Transfer learning reduces the number of labeled images required for training by more than 10 times:

<img src="./assets/Training performance.svg">

To use transfer learning in training, add the following to the configuration:

```json
{
  "transfer_learning_base": "models/transfer_learning_base/features.tf",
  "pad_to": 32
} 
```
The input shape must be divisible by 32, for example:

```json
{
  "input_shape": [
    384,
    384,
    3
  ]
} 
```

Then train on your dataset to make the fine-tuning of the base model.

## Rotation-invariant object detection

If all you need is to adapt the detection model to object rotation instead of finding the orientation, you can do 
the following:

1. Label object positions only, with arbitrary rotation angles.
2. In the configuration file, specify:
```json
{
  "loss_weight_angle": 0,
  "angle_diff_thr": 999
} 
``` 
