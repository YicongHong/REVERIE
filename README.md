# REVERIE: Remote Embodied Visual Referring Expression in Real Indoor Environments
+ New: The evaluation server is online here. To prepare submission files, please see the [Result file format](#resformat) below.

Here are the pre-released code and data for the CVPR 2020 paper [REVERIE: Remote Embodied Visual Referring Expression in Real Indoor Environments](https://arxiv.org/abs/1904.10151)
<div align="center">
<img src="REVERIE_task.png" width = "300" height = "300" alt="REVERIE task example" align=center />
</div>

## 1. Definition of the REVERIE Task
As shown in the above figure, a robot agent is given a natural language instruction referring to a remote object (here in the red bounding box) in a photo-realistic 3D environment. The agent must navigate to an appropriate location and identify the object from multiple distracting candidates. The blue discs indicate nearby navigable viewpoints provided the simulator.

## 2. Install without Docker

**Note** This section prepares everything to run or train our Navigator-Pointer model. If you are familar with R2R and just want to do the REVERIE task, you can jump this section and directly go to xxx.
**Note** If you have a fresh Ubuntu system, the following install should work well. If not, you can still try this manner but it may screw up your existing project environments. Finnaly, you can try the installation with docker.

### Prerequisites

A C++ compiler with C++11 support is required. Matterport3D Simulator has several dependencies:
- [Ubuntu](https://www.ubuntu.com/) 14.04, 16.04, 18.04
- [OpenCV](http://opencv.org/) >= 2.4 including 3.x 
- [OpenGL](https://www.opengl.org/)
- [OSMesa](https://www.mesa3d.org/osmesa.html)
- [GLM](https://glm.g-truc.net/0.9.8/index.html)
- [Numpy](http://www.numpy.org/)
- [pybind11](https://github.com/pybind/pybind11) for Python bindings
- [Doxygen](http://www.doxygen.org) for building documentation

E.g. installing dependencies on Ubuntu:
```
sudo apt-get install libopencv-dev python-opencv freeglut3 freeglut3-dev libglm-dev libjsoncpp-dev doxygen libosmesa6-dev libosmesa6 libglew-dev
```

### 2.1. Clone Repo
Clone the Self-Monitoring Agent repository:
```
# Make sure to clone with --recursive
git clone https://github.com/YuankaiQi/REVERIE.git
cd REVERIE
```

Note that our repository is based on the v0.1 version [Matterport3DSimulator](https://github.com/peteanderson80/Matterport3DSimulator), which was originally proposed with the Room-to-Roon dataset. 

### 2.2. MAttNet3 Download
Download the our pre-trained mini MAttnet3 from <a href="https://drive.google.com/uc?id=1kxyH8kWHiskbv7bRzYzC9rM1lX51K004&export=download" target="_blank">Google Drive</a> or <a href="https://pan.baidu.com/s/1Y5KoAV4FyrFz-KzMBsZmBw" target="_blank">Baidu Yun, code:ophy</a>, which is modified from <a href="https://github.com/lichengunc/MAttNet" target="_blank">MAttNet</a> to support our model training. Unzip it into the MAttnet3 folder. This is used as the our Pointer model.

### 2.3. Dataset Download
You need to download RGB images and house segmentation files of the [Matterport3D dataset](https://niessner.github.io/Matterport/). The following data types are required:
- `matterport_skybox_images`
- `house_segmentations`
Then set the root path, e.g. /home/qyk/dataset/Matterport/, to the 'matterportDir' in trainFast.py.

### 2.4. Pre-computed Image Features Download
Download and extract the tsv files into the `img_features` directory from [Matterport3DSimulator](https://github.com/peteanderson80/Matterport3DSimulator). You will only need the ImageNet features to replicate our results. 
- [ResNet-152-imagenet features [380K/2.9GB]](https://www.dropbox.com/s/715bbj8yjz32ekf/ResNet-152-imagenet.zip)

### 2.5. Installation with PyTorch
Let us get things ready to run experiments.

#### 2.5.1. Create Anaconda enviorment
```bash
# change "rog" (remote object grounding) to any name you prefer
conda create -n rog python=3.6
```
Activate the enviorment you just created
```
conda activate rog
```
#### 2.5.2. Install special requirements
```
pip install -r tasks/REVERIE/requirements.txt
```

#### 2.5.3. Install PyTorch
Check the official [PyTorch website](http://pytorch.org/) for different CUDA version.
```
# with CUDA 92
conda install pytorch=0.4.0 cuda92 -c pytorch
conda install torchvision=0.1.8 -c pytorch
```

### 2.6 Compile the Matterport3D Simulator
Let us compile the simulator so that we can call its functions in python.

Build OpenGL version using CMake:
```bash
cd build
cmake ..

# Double-check if CMake find the proper path to your python
# if not, remove the make files and use the cmake with option below instead
rm -rf *
cmake -DPYTHON_EXECUTABLE:FILEPATH=/path/to/your/bin/python ..

make
cd ../
```
Or build headless OSMESA version using CMake:
```
mkdir build && cd build
cmake -DOSMESA_RENDERING=ON ..
make
cd ../
```

### 2.7 Compile MAttNet3
#### 2.7.1. Compile pytorch-faster-rcnn
```
cd MAttNet3/pyutils/mask-faster-rcnn/lib
```
You may need to change the `-arch` version in `Makefile` to compile the cuda code:

  | GPU model  | Architecture |
  | ------------- | ------------- |
  | TitanX (Maxwell/Pascal) | sm_52 |
  | GTX 960M | sm_50 |
  | GTX 1080 (Ti) | sm_61 |
  | Grid K520 (AWS g2.2xlarge) | sm_30 |
  | Tesla K80 (AWS p2.xlarge) | sm_37 |

Compile the CUDA-based `nms` and `roi_pooling` using following simple commands:
```
make
```

#### 2.7.2. Compile refer
```
cd ../../refer
make
```
It will generate ``_mask.c`` and ``_mask.so`` in ``external/`` folder.

## 3. Install with Docker (not tested)

### 3.1. Prerequisites

- Nvidia GPU with driver >= 396.37
- Install [docker](https://docs.docker.com/engine/installation/)
- Install [nvidia-docker2.0](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0))
- Note: CUDA / CuDNN toolkits do not need to be installed (these are provided by the docker image)

### 3.2 Clone Repo

Clone the Matterport3DSimulator repository:
```
# Make sure to clone with --recursive
git clone --recursive https://github.com/peteanderson80/Matterport3DSimulator.git
cd Matterport3DSimulator
```

If you didn't clone with the `--recursive` flag, then you'll need to manually clone the pybind submodule from the top-level directory:
```
git submodule update --init --recursive
```

### 3.3. Dataset Download

See [Section 2.3](#2.3-Dataset-Download)

Set an environment variable to the location of the dataset, where <PATH> is the full absolute path (not a relative path or symlink) to the directory containing the individual matterport scan directories (17DRP5sb8fy, 2t7WUuJeko7, etc):
```
export MATTERPORT_DATA_DIR=<PATH>
```

Note that if <PATH> is a remote sshfs mount, you will need to mount it with the `-o allow_root` option or the docker container won't be able to access this directory. 

### Dataset Preprocessing

To make data loading faster and to reduce memory usage we preprocess the `matterport_skybox_images` by downscaling and combining all cube faces into a single image using the following script:
```
./scripts/downsize_skybox.py
```

This will take a while depending on the number of processes used. By default images are downscaled by 50% and 20 processes are used.

## 4. Train and Test the model
+ **For training**

You can download our pre-trained models from [Google Drive](https://drive.google.com/uc?id=16Kj1L3m7QWSffzygnuue084Q9b7IOhsQ&export=download) or [Baidu Yun](). If you want to train by yourself, just run the following command:
```
python tasks/REVERIE/trainFast.py --feedback_method sample2step --experiment_name releaseCheck
```
+ **For testing**
To test the model, you need first obtain navigation results by 
```
python tasks/REVERIE/run_search.py
```
then run the following command to obtain the grounded object
```
python tasks/REVERIE/groundingAfterNav.py
```
Now, you should get results in the 'experiment/releaseCheck/results/' folder.

Note that the results might be slightly different due to using different dependant package versions or GPUs. 

## 5. Data Organization of the REVERIE task
Unzip the data.zip and bbox.zip files. Then in the data folder, you get REVERIE_train.json, REVERIE_val_seen.json, and REVERIE_val_unseen.json three files, which provide instructions, paths, and target object of each task. In the bbox folder, you get json files that record objects observed at each viewpoint within 3 meters.

+ **Example of tarin/val_seen/val_unseen.json file**
```
[
  {
    "distance" : 11.65, # distance to the goal viewpoint
    "ix": 208,  # Reserved data, not used
    "scan": "qoiz87JEwZ2", # building ID
    "heading": 4.59, # initial parameters for agent
    "path_id": 1357, # inherited from the R2R dataset
    "objId": 66, # the unique object id in the current building 
    "id": "1357_66" # task id
    "instructions":[ # collected instructions for REVERIE
        "Go to the entryway and clean the coffee table", 
        "Go to the foyer and wipe down the coffee table", 
        "Go to the foyer on level 1 and pull out the coffee table further from the chair"
     ]
    "path": [ # inherited from the R2R dataset
        "bdb1023cb7cc4ebd8245b9291fcbc1a2", 
        "a6ba3f53b7964464b23341896d3c75fa", 
        "c407e34577aa4724b7e5d447a5d859d1", 
        "9f68b19f50d14f5d8371447f73c3a2e3", 
        "150763c717894adc8ccbbbe640fa67ef", 
        "59b190857cfe47f691bf0d866f1e5aeb", 
        "267a7e2459054db7952fc1e3e45e98fa"
      ]
     "instructions_l":[ # inherited from the R2R dataset and provided just for convenience 
        "Walk into the dining room and continue past the table. Turn left when you xxx ", 
       ...
       ]
  },
  ...
]
```
+ **Example of json file in the bbox folder**

    File name format: ScanID_ViewpointID.json, e.g.,VzqfbhrpDEA_57fba128d2f042f7a59793c665a3f587.json
```
{ # note that this is in the type of dict not list
  "57fba128d2f042f7a59793c665a3f587":{ # this key is the id of viewpoint
    "827":{ # this key is the id of object 
      "name": "toilet",
      "visible_pos":[
        6,7,8,9,19,20  # these are view index (0~35) which contain the object. Index is consitent with that in  R2R 
        ],
      "bbox2d":[
        [585,382,55,98], # [x,y,w,h] and corresponds to the views listed in the "visible_pos"
        ...
       ]
    },
    "833": {
       ...
    },
    ...
  }
}
```
## 4. Integrate into Your Project

The easiest way to integrate these object infomation into your project is to preload all the objects bounding box/label/visible_pos with the **loadObjProposals()** function as in the eval.py file. Then you can access visible objects using ScanID_ViewpointID as key. You can use any referring expression methods to get matched objects with an instruction.

## Result File Format

Just add the "'predObjId': int value" pair into your navigation results. That's it!

Below is a toy sample:

```
[
  {
    "trajectory": [
      [
        "a68b5ae6571e4a66a4727573b88227e4", 
        3.141592653589793, 
        0.0
      ], 
      ...
     ],
     "instr_id": "4774_267_1", 
     "predObjId": 402
  },
  ...
]
```

## Note
+ We modify the method to load dataset, see **load_datasets_REVERIE()** in utils.py
+ The number of instructions may vary across the dataset, we recommend the following way to index an instruction:
```
instrType = "instructions"
self.instr_ids += ['%s_%d' % (str(item['id']),i) for i in range(len(item[instrType]))]

```
+ To get the Remote Grounding Success Rate in the eval.py file, you need to implement the method to read the predicted object id from your results file. And then compare it against the 'objId' in the \_score_item() function.
## Acknowledgements
We would like to thank Matterport for allowing the Matterport3D dataset to be used by the academic community. This project is supported by the [Australian Centre for Robotic Vision](https://www.roboticvision.org/). We also thank [Philip Roberts](mailto:philip.roberts@adelaide.edu.au), [Zheng Liu](mailto:marco.liu19@imperial.ac.uk), and [Zizheng Pan](mailto:zizheng.pan@student.adelaide.edu.au), and [Sam Bahrami](https://www.roboticvision.org/rv_person/sam-bahrami/) for their great help in building the dataset.
## Reference
The REVERIE task and dataset are descriped in:
```
@inproceedings{reverie,
  title={REVERIE: Remote Embodied Visual Referring Expression in Real Indoor Environments},
  author={Yuankai Qi and Qi Wu and Peter Anderson and Xin Wang and William Yang Wang and Chunhua Shen and Anton van den Hengel},
  booktitle={Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
  year={2020}
}

```
