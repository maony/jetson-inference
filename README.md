<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/deep-vision-header.jpg">

# Deploying Deep Learning
Welcome to our guide to deploying inference and embedded [deep vision](https://rawgit.com/dusty-nv/jetson-inference/master/docs/html/index.html) runtime library for **[NVIDIA Jetson TX1/TX2](http://www.nvidia.com/object/embedded-systems.html)**.

Included in this repo are resources for efficiently deploying neural networks into the field using NVIDIA **[TensorRT](https://developer.nvidia.com/tensorrt)**.

Vision primitives, such as [`imageNet`](imageNet.h) for image recognition, [`detectNet`](detectNet.h) for object localization, and [`segNet`](segNet.h) for segmentation, inherit from the shared [`tensorNet`](tensorNet.h) object.  Examples are provided for streaming from live camera feed and processing images from disk.  See the **[Deep Vision API Reference Specification](https://rawgit.com/dusty-nv/jetson-inference/master/docs/html/index.html)** for documentation accompanying this tutorial. 

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/deep-vision-primitives.png" width="800">

> ![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/new.jpg) Read our recent **[Parallel ForAll post](https://devblogs.nvidia.com/parallelforall/jetson-tx2-delivers-twice-intelligence-edge/)**, *NVIDIA Jetson TX2 Delivers Twice the Intelligence to the Edge*. <br/>
> ![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/new.jpg) Support for **[Image Segmentation](#image-segmentation-with-segnet)** models and training guide with aerial drone dataset.

### **Table of Contents**

* [DIGITS Workflow](#digits-workflow) 
* [System Setup](#system-setup)
* [Building from Source on Jetson](#building-from-source-on-jetson)
* [Classifying Images with ImageNet](#classifying-images-with-imagenet)
	* [Using the Console Program](#using-the-console-program)
	* [Running the Live Camera Recognition Demo](#running-the-live-camera-recognition-demo)
	* [Re-training the Network with DIGITS](#re-training-the-network-with-DIGITS)
* [Locating Object Coordinates using DetectNet](#locating-object-coordinates-using-detectNet)
	* [Testing Images from the Command Line](#testing-images-from-the-command-line)
	* [Multi-class Object Detection](#multi-class-object-detection)
	* [Running the Live Camera Detection Demo](#running-the-live-camera-detection-demo)
	* [Re-training DetectNet with DIGITS](#re-training-detectnet-with-digits)
* [Image Segmentation with SegNet](#image-segmentation-with-segnet)
	* [Downloading Aerial Drone Dataset](#downloading-aerial-drone-dataset)
	* [Importing the Dataset into DIGITS](#importing-the-dataset-into-digits)
	* [Generating Pretrained FCN-Alexnet](#generating-pretrained-fcn-alexnet)
	* [Training FCN-Alexnet with DIGITS](#training-fcn-alexnet-with-digits)
	* [Testing Inference Model in DIGITS](#testing-inference-model-in-digits)
	* [FCN-Alexnet Patches for TensorRT](#fcn-alexnet-patches-for-tensorrt)
	* [Running Segmentation Models on Jetson](#running-segmentation-models-on-jetson)
* [Extra Resources](#extra-resources)

**Recommended System Requirements**

Training GPU:  Maxwell or Pascal-based GPU or AWS P2 instance.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ubuntu 14.04 x86_64 or Ubuntu 16.04 x86_64 (see DIGITS [AWS AMI](https://aws.amazon.com/marketplace/pp/B01LZN28VD) image).

Deployment:    &nbsp;&nbsp;Jetson TX2 Developer Kit with JetPack 3.0 or newer (Ubuntu 16.04 aarch64).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Jetson TX1 Developer Kit with JetPack 2.3 or newer (Ubuntu 16.04 aarch64).

> **note**:  this [branch](http://github.com/dusty-nv/jetson-inference) is verified against the following BSP versions for Jetson TX1/TX2: <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;> Jetson TX2 - JetPack 3.0 / L4T R27.1 aarch64 (Ubuntu 16.04 LTS) <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;> Jetson TX1 - JetPack 2.3 / L4T R24.2 aarch64 (Ubuntu 16.04 LTS) <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;> Jetson TX1 - JetPack 2.3.1 / L4T R24.2.1 aarch64 (Ubuntu 16.04 LTS)

Note that TensorRT samples from the repo are intended for deployment on embedded Jetson TX1/TX2 module, however when cuDNN and TensorRT have been installed on the host side, the TensorRT samples in the repo can be compiled for PC.

## DIGITS Workflow

New to deep neural networks (DNNs) and machine learning?  Take this [introductory primer](docs/deep-learning.md) on training and inference.

<a href="https://github.com/dusty-nv/jetson-inference/blob/master/docs/deep-learning.md"><img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/digits-samples.jpg" width="800"></a>

Using NVIDIA deep learning tools, it's easy to **[Get Started](https://github.com/NVIDIA/DIGITS/blob/master/docs/GettingStarted.md)** training DNNs and deploying them into the field with high performance.  Discrete GPUs are typically used in a server, PC, or laptop for training with DIGITS, while Jetson and integrated GPU is used by embedded form factors.

<a href="https://github.com/dusty-nv/jetson-inference/blob/master/docs/deep-learning.md"><img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/digits-workflow.jpg" width="700"></a>

NVIDIA [DIGITS](https://github.com/NVIDIA/DIGITS) is used to interactively train network models on annotated datasets in the cloud or PC, while TensorRT and Jetson are used to deploy runtime inference in the field. TensorRT uses graph optimizations and half-precision FP16 support to more than double DNN inferencing.  Together, DIGITS and TensorRT form an effective workflow for developing and deploying deep neural networks capable of implementing advanced AI and perception. 

## System Setup

During this tutorial, we will use a host PC (or AWS), for training DNNs, alongside a Jetson for inference.  The host PC will also serve to flash the Jetson with the latest JetPack.  First we'll setup and configure the host PC with the required OS and tools.

### Installing Ubuntu on the Host

Download and install Ubuntu 16.04 x86_64 onto the host PC from one of the following locations:

```
http://releases.ubuntu.com/16.04/ubuntu-16.04.2-desktop-amd64.iso
http://releases.ubuntu.com/16.04/ubuntu-16.04.2-desktop-amd64.iso.torrent
```

Ubuntu 14.04 x86_64 may also be acceptable with minor modifications later while installing some packages with apt-get.

### Running JetPack on the Host

Download the latest **[JetPack](https://developer.nvidia.com/embedded/jetpack)** to the host PC.  In addition to flashing the Jetson with the latest Board Support Package (BSP), JetPack automatically installs tools for the host like CUDA Toolkit.  See the JetPack [Release Notes](https://developer.nvidia.com/embedded/jetpack-notes) for the full list of features and installed packages.

After downloading JetPack from the link above, run it from the host PC with the following commands:

``` bash 
$ cd <directory where you downloaded JetPack>
$ chmod +x JetPack-L4T-3.0-linux-x64.run 
$ ./JetPack-L4T-3.0-linux-x64.run 
```

The JetPack GUI will start.  Follow the step-by-step **[Install Guide](http://docs.nvidia.com/jetpack-l4t/index.html#developertools/mobile/jetpack/l4t/3.0/jetpack_l4t_install.htm)** to complete the setup.  Near the beginning, JetPack will confirm which generation Jetson you are developing for.

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/jetpack-platform.png" width="450">

Select Jetson TX1 if you are using TX1, or Jetson TX2 if you're using TX2, and press `Next` to continue.

The next screen will list the packages available to be installed.  The packages installed to the host are listed at the top under the `Host - Ubuntu` dropdown, while those intended for the Jetson are shown near the bottom.  You can select or deselect an individual package for installation by clicking it's `Action` column.

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/jetpack-downloads.png" width="500">

Since CUDA will be used on the host for training DNNs, it's recommended to select the Full install by click on the radio button in the top right.  Then press `Next` to begin setup.  JetPack will download and then install the sequence of packages.  Note that all the .deb packages are stored under the `jetpack_downloads` subdirectory if you are to need them later.  

After the downloads have finished installing, JetPack will enter the post-install phase where the JetPack is flashed with the L4T BSP.  You'll need to connect your Jetson to your host PC via the micro-USB port and cable included in the devkit.  Then enter your Jetson into recovery mode by holding down the Recovery button while pressing and releasing Reset.  If you type `lsusb` from the host PC after you've connected the micro-USB cable and entered the Jetson into recovery mode, you should see the NVIDIA device come up under the list of USB devices.  JetPack uses the micro-USB connection from the host to flash the L4T BSP to the Jetson.  After flashing, the Jetson will reboot and if attached to an HDMI display, will boot up to the Ubuntu desktop.  After this, JetPack connects to the Jetson from the host via SSH to install additional packages to the Jetson, like the ARM aarch64 builds of CUDA Toolkit, cuDNN, and TensorRT.  For JetPack to be able to reach the Jetson via SSH, the host PC should be networked to the Jetson via Ethernet.  This can be accomplished by running an Ethernet cable directly from the host to the Jetson, or by connecting both devices to a router or switch — the JetPack GUI will ask you to confirm which networking scenario is being used.  See the JetPack **[Install Guide](http://docs.nvidia.com/jetpack-l4t/index.html#developertools/mobile/jetpack/l4t/3.0/jetpack_l4t_install.htm)** for the full directions for installing JetPack and flashing Jetson.

### Installing NVIDIA Driver on the Host

At this point, JetPack will have flashed the Jetson with the latest L4T BSP, and installed CUDA toolkits to both the Jetson and host PC.  However, the NVIDIA PCIe driver will still need to be installed on the host PC to enable GPU-accelerated training.  Run the following commands from the host PC to install the NVIDIA driver from the Ubuntu repo:

``` bash
$ sudo apt-get install nvidia-375
$ sudo reboot
```

Afer rebooting, the NVIDIA driver should be listed under `lsmod`:

``` bash
$ lsmod | grep nvidia
nvidia_uvm            647168  0
nvidia_drm             49152  1
nvidia_modeset        790528  4 nvidia_drm
nvidia              12144640  60 nvidia_modeset,nvidia_uvm
drm_kms_helper        167936  1 nvidia_drm
drm                   368640  4 nvidia_drm,drm_kms_helper
```

To verify the CUDA toolkit and NVIDIA driver are working, run some tests that come with the CUDA samples:

``` bash
$ cd /usr/local/cuda/samples
$ sudo make
$ cd bin/x86_64/linux/release/
$ ./deviceQuery
$ ./bandwidthTest --memory=pinned
```

### Installing cuDNN on the Host

The next step is to install NVIDIA **[cuDNN](https://developer.nvidia.com/cudnn)** libraries on the host PC.  Download the libcudnn and libcudnn packages from the NVIDIA site:

```
https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v6/prod/8.0_20170307/Ubuntu16_04_x64/libcudnn6_6.0.20-1+cuda8.0_amd64-deb
https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v6/prod/8.0_20170307/Ubuntu16_04_x64/libcudnn6-dev_6.0.20-1+cuda8.0_amd64-deb
```

Then install the packages with the following commands:

``` bash
$ sudo dpkg -i libcudnn6_6.0.20-1+cuda8.0_amd64.deb
$ sudo dpkg -i libcudnn6-dev_6.0.20-1+cuda8.0_amd64.deb
```

### Installing NVcaffe on the Host

NVcaffe is the NVIDIA branch of Caffe with optimizations for GPU.  NVcaffe uses cuDNN and is used by DIGITS for training DNNs.  To install it, clone the NVcaffe repo from GitHub and compile from source.  First some prequisite packages for Caffe are installed, including the Python bindings required by DIGITS:

``` bash
$ sudo apt-get install --no-install-recommends build-essential cmake git gfortran libatlas-base-dev libboost-filesystem-dev libboost-python-dev libboost-system-dev libboost-thread-dev libgflags-dev libgoogle-glog-dev libhdf5-serial-dev libleveldb-dev liblmdb-dev libprotobuf-dev libsnappy-dev protobuf-compiler python-all-dev python-dev python-h5py python-matplotlib python-numpy python-opencv python-pil python-pip python-protobuf python-scipy python-skimage python-sklearn python-setuptools 
$ sudo pip install --upgrade pip
$ git clone http://github.com/NVIDIA/caffe
$ cd caffe
$ sudo pip install -r python/requirements.txt 
$ mkdir build
$ cd build
$ cmake ../ -DUSE_OPENCV=off
$ make --jobs=4
$ make pycaffe
```

Caffe should now be configured and built.  Now edit your user's ~/.bashrc to include the path to your Caffe tree (replace the paths below to reflect your own):

``` bash
export CAFFE_ROOT=/home/dusty/workspace/caffe
export PYTHONPATH=/home/dusty/workspace/caffe/python:$PYTHONPATH
```

Close and re-open the terminal for the changes to take effect.


### Installing DIGITS on the Host

NVIDIA **[DIGITS](https://developer.nvidia.com/digits)** is a Python-based web service which interactively trains DNNs and manages datasets.  As highlighed in the DIGITS workflow, it runs on the host PC to create the network model during the training phase.  The trained model is then copied from the host PC to the Jetson for the runtime inference phase with TensorRT.

To install DIGITS, first install the prerequisiste packages, then clone the DIGITS repo from GitHub:

``` bash
$ sudo apt-get install --no-install-recommends graphviz python-dev python-flask python-flaskext.wtf python-gevent python-h5py python-numpy python-pil python-pip python-protobuf python-scipy python-tk
$ git clone http://github.com/nvidia/DIGITS
$ cd DIGITS
$ sudo pip install -r requirements.txt
```

#### Starting the DIGITS Server

Assuming that your terminal is still in the DIGITS directory, the webserver can be started by running the `digits-devserver` Python script:

``` bash
$ ./digits-devserver 
  ___ ___ ___ ___ _____ ___
 |   \_ _/ __|_ _|_   _/ __|
 | |) | | (_ || |  | | \__ \
 |___/___\___|___| |_| |___/ 5.1-dev

2017-04-17 13:19:02 [INFO ] Loaded 0 jobs.
```

DIGITS will store user jobs (training datasets and model snapshots) under the `digits/jobs` directory.

To access the interactive DIGITS session, open your web browser and navigate to `0.0.0.0:5000`.

> **note**:  by default the DIGITS server will start on port 5000, but the port can be specified by passing the `--port` argument to the `digits-devserver` script.


## Building from Source on Jetson
Provided along with this repo are TensorRT-enabled deep learning primitives for running Googlenet/Alexnet on live camera feed for image recognition, pedestrian detection networks with localization capabilities (i.e. that provide bounding boxes), and segmentation.  This repo is intended to be built & run on the Jetson and to accept the network models from the host PC trained on the DIGITS server.

The latest source can be obtained from [GitHub](http://github.com/dusty-nv/jetson-inference) and compiled onboard Jetson TX1/TX2.

> **note**:  this [branch](http://github.com/dusty-nv/jetson-inference) is verified against the following BSP versions for Jetson TX1/TX2: <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;> Jetson TX2 - JetPack 3.0 / L4T R27.1 aarch64 (Ubuntu 16.04 LTS) <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;> Jetson TX1 - JetPack 2.3 / L4T R24.2 aarch64 (Ubuntu 16.04 LTS) <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;> Jetson TX1 - JetPack 2.3.1 / L4T R24.2.1 aarch64 (Ubuntu 16.04 LTS)
      
#### Cloning the Repo
To obtain the repository, navigate to a folder of your choosing on the Jetson.  First, make sure git and cmake are installed locally:

``` bash
$ sudo apt-get install git cmake
```

Then clone the jetson-inference repo:
``` bash
$ git clone http://github.com/dusty-nv/jetson-inference
```

#### Configuring with CMake

When cmake is run, a special pre-installation script (CMakePreBuild.sh) is run and will automatically install any dependencies.

``` bash
$ cd jetson-inference
$ mkdir build
$ cd build
$ cmake ../
```

#### Compiling the Project

Make sure you are still in the jetson-inference/build directory, created above in step #2.

``` bash
$ cd jetson-inference/build			# omit if pwd is already /build from above
$ make
```

Depending on architecture, the package will be built to either armhf or aarch64, with the following directory structure:

```
|-build
   \aarch64		    (64-bit)
      \bin			where the sample binaries are built to
      \include		where the headers reside
      \lib			where the libraries are build to
   \armhf           (32-bit)
      \bin			where the sample binaries are built to
      \include		where the headers reside
      \lib			where the libraries are build to
```

binaries residing in aarch64/bin, headers in aarch64/include, and libraries in aarch64/lib.

#### Digging Into the Code

For reference, see the available vision primitives, including [`imageNet`](imageNet.h) for image recognition and [`detectNet`](detectNet.h) for object localization.

``` c++
/**
 * Image recognition with GoogleNet/Alexnet or custom models, using TensorRT.
 */
class imageNet : public tensorNet
{
public:
	/**
	 * Network choice enumeration.
	 */
	enum NetworkType
	{
		ALEXNET,
		GOOGLENET
	};

	/**
	 * Load a new network instance
	 */
	static imageNet* Create( NetworkType networkType=GOOGLENET );
	
	/**
	 * Load a new network instance
	 * @param prototxt_path File path to the deployable network prototxt
	 * @param model_path File path to the caffemodel
	 * @param mean_binary File path to the mean value binary proto
	 * @param class_info File path to list of class name labels
	 * @param input Name of the input layer blob.
	 */
	static imageNet* Create( const char* prototxt_path, const char* model_path, const char* mean_binary,
							 const char* class_labels, const char* input="data", const char* output="prob" );

	/**
	 * Determine the maximum likelihood image class.
	 * @param rgba float4 input image in CUDA device memory.
	 * @param width width of the input image in pixels.
	 * @param height height of the input image in pixels.
	 * @param confidence optional pointer to float filled with confidence value.
	 * @returns Index of the maximum class, or -1 on error.
	 */
	int Classify( float* rgba, uint32_t width, uint32_t height, float* confidence=NULL );
};
```

Both inherit from the shared [`tensorNet`](tensorNet.h) object which contains common TensorRT code.

## Classifying Images with ImageNet
There are multiple types of deep learning networks available, including recognition, detection/localization, and soon segmentation.  The first deep learning capability we're highlighting in this tutorial is **image recognition** using an 'imageNet' that's been trained to identify similar objects.

The [`imageNet`](imageNet.h) object accepts an input image and outputs the probability for each class.  Having been trained on ImageNet database of **[1000 objects](data/networks/ilsvrc12_synset_words.txt)**, the standard AlexNet and GoogleNet networks are downloaded during [step 2](#configuring-with-cmake) from above.  As examples of using [`imageNet`](imageNet.h) we provide a command-line interface called [`imagenet-console`](imagenet-console/imagenet-console.cpp) and a live camera program called [`imagenet-camera`](imagenet-camera/imagenet-camera.cpp).

### Using the Console Program

First, use the [`imagenet-console`](imagenet-console/imagenet-console.cpp) program to test imageNet recognition on some example images. After [building](#building-from-source-on-jetson), make sure your terminal is located in the aarch64/bin directory:

``` bash
$ cd jetson-inference/build/aarch64/bin
```

Then, classify an example image with the [`imagenet-console`](imagenet-console/imagenet-console.cpp) program.  [`imagenet-console`](imagenet-console/imagenet-console.cpp) accepts 2 command-line arguments:  the path to the input image and path to the output image (with the class overlay printed).

``` bash
$ ./imagenet-console orange_0.jpg output_0.jpg
```

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/imagenet-orange.jpg" width="700">

``` bash
$ ./imagenet-console granny_smith_1.jpg output_1.jpg
```

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/imagenet-apple.jpg" width="700">

Next, we will use [imageNet](imageNet.h) to classify a live video feed from the Jetson onboard camera.

### Running the Live Camera Recognition Demo

Similar to the last example, the realtime image recognition demo is located in /aarch64/bin and is called [`imagenet-camera`](imagenet-camera/imagenet-camera.cpp).
It runs on live camera stream and depending on user arguments, loads googlenet or alexnet with TensorRT. 
``` bash
$ ./imagenet-camera googlenet           # to run using googlenet
$ ./imagenet-camera alexnet             # to run using alexnet
```

The frames per second (FPS), classified object name from the video, and confidence of the classified object are printed to the openGL window title bar.  By default the application can recognize up to 1000 different types of objects, since Googlenet and Alexnet are trained on the ILSVRC12 ImageNet database which contains 1000 classes of objects.  The mapping of names for the 1000 types of objects, you can find included in the repo under [data/networks/ilsvrc12_synset_words.txt](http://github.com/dusty-nv/jetson-inference/blob/master/data/networks/ilsvrc12_synset_words.txt)

> **note**:  by default, the Jetson's onboard CSI camera will be used as the video source.  If you wish to use a USB webcam instead, change the `DEFAULT_CAMERA` define at the top of [`imagenet-camera.cpp`](imagenet-camera/imagenet-camera.cpp) to reflect the /dev/video V4L2 device of your USB camera.  The model it's tested with is Logitech C920. 

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/imagenet-orange-camera.jpg" width="800">
<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/imagenet-apple-camera.jpg" width="800">

### Re-training the Network with DIGITS

The existing GoogleNet and AlexNet models that are downloaded by the repo are pre-trained on [1000 classes of objects](data/networks/ilsvrc12_synset_words.txt).

What if you require a new object class to be added to the network, or otherwise require a different organization of the classes?  

Using [NVIDIA DIGITS](http://github.com/NVIDIA/DIGITS), networks can be fine-tuned or re-trained from a pre-exisiting network model.
After installing DIGITS on a PC or in the cloud (such as an AWS instance), see the **[Image Folder Specification](https://github.com/NVIDIA/DIGITS/blob/master/docs/ImageFolderFormat.md)** to learn how to organize the data for your particular application.

Popular training databases with various annotations and labels include [ImageNet](image-net.org), [MS COCO](mscoco.org), and [Google Images](images.google.com) among others.

See [here](http://www.deepdetect.com/tutorials/train-imagenet/) under the `Downloading the dataset` section to obtain a crawler script that will download the 1000 original classes, including as many of the original images that are still available online.

> **note**: be considerate running the crawler script from a corporate network, they may flag the activity.
> It will probably take overnight on a decent connection to download the 1000 ILSVRC12 classes (100GB) from ImageNet (1.2TB)

Then, while creating the new network model in DIGITS, copy the [GoogleNet prototxt](data/networks/googlenet.prototxt) and specify the existing GoogleNet caffemodel as the DIGITS **Pretrained Model**:

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/imagenet-digits-prototxt.jpg" width="800">

The network training should now converge faster than if it were trained from scratch.  After the desired accuracy has been reached, copy the new model checkpoint back over to your Jetson and proceed as before, but now with the added classes available for recognition.

## Locating Object Coordinates using DetectNet
The previous image recognition examples output class probabilities representing the entire input image.   The second deep learning capability we're highlighting in this tutorial is detecting multiple objects, and finding where in the video those objects are located (i.e. extracting their bounding boxes).  This is performed using a 'detectNet' - or object detection / localization network.

The [`detectNet`](detectNet.h) object accepts as input the 2D image, and outputs a list of coordinates of the detected bounding boxes.  Three example detection network models are are automatically downloaded during the repo [source configuration](#configuring):

1. **ped-100**  (single-class pedestrian detector)
2. **multiped-500**   (multi-class pedestrian + baggage detector)
3. **facenet-120**  (single-class facial recognition detector)

As with the previous examples, provided are a console program and a camera streaming program for using detectNet.

### Processing Images from the Command Line

To process test images with [`detectNet`](detectNet.h) and TensorRT, use the [`detectnet-console`](detectnet-console/detectnet-console.cpp) program.  [`detectnet-console`](detectnet-console/detectnet-console.cpp) accepts command-line arguments representing the path to the input image and path to the output image (with the bounding box overlays rendered).  Some test images are included with the repo:

``` bash
$ ./detectnet-console peds-007.png output-7.png
```

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/detectnet-peds-00.jpg" width="900">

To change the network that [`detectnet-console`](detectnet-console/detectnet-console.cpp) uses, modify [`detectnet-console.cpp`](detectnet-console/detectnet-console.cpp) (beginning line 33):
``` c
detectNet* net = detectNet::Create( detectNet::PEDNET_MULTI );	 // uncomment to enable one of these 
//detectNet* net = detectNet::Create( detectNet::PEDNET );
//detectNet* net = detectNet::Create( detectNet::FACENET );
```
Then to recompile, navigate to the `jetson-inference/build` directory and run `make`.

### Multi-class Object Detection
When using the multiped-500 model (`PEDNET_MULTI`), for images containing luggage or baggage in addition to pedestrians, the 2nd object class is rendered with a green overlay.
``` bash
$ ./detectnet-console peds-008.png output-8.png
```

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/detectnet-peds-01.jpg" width="900">

### Running the Live Camera Detection Demo

Similar to the previous example, [`detectnet-camera`](detectnet-camera/detectnet-camera.cpp) runs the object detection networks on live video feed from the Jetson onboard camera.  Launch it from command line along with the type of desired network:

``` bash
$ ./detectnet-camera multiped       # run using multi-class pedestrian/luggage detector
$ ./detectnet-camera ped-100        # run using original single-class pedestrian detector
$ ./detectnet-camera facenet        # run using facial recognition network
$ ./detectnet-camera                # by default, program will run using multiped
```

> **note**:  to achieve maximum performance while running detectnet, increase the Jetson TX1 clock limits by running the script:
>  `sudo ~/jetson_clocks.sh`
<br/>
> **note**:  by default, the Jetson's onboard CSI camera will be used as the video source.  If you wish to use a USB webcam instead, change the `DEFAULT_CAMERA` define at the top of [`detectnet-camera.cpp`](detectnet-camera/detectnet-camera.cpp) to reflect the /dev/video V4L2 device of your USB camera.  The model it's tested with is Logitech C920.  

### Re-training DetectNet with DIGITS

For a step-by-step guide to training custom DetectNets, see the **[Object Detection](https://github.com/NVIDIA/DIGITS/tree/digits-4.0/examples/object-detection)** example included in DIGITS version 4:

<a href="https://github.com/NVIDIA/DIGITS/tree/digits-4.0/examples/object-detection"><img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/detectnet-digits-overview.jpg" width="500"></a>

The DIGITS guide above uses the [KITTI](http://www.cvlibs.net/datasets/kitti/) dataset, however [MS COCO](http://mscoco.org) also has bounding data available for a variety of objects.

## Image Segmentation with SegNet

The third deep learning capability we're highlighting in this tutorial is image segmentation.  Segmentation is based on image recognition, except the classifications occur at the pixel level as opposed to classifying entire images as with image recognition.  This is accomplished by *convolutionalizing* a pre-trained imageNet recognition model (like Alexnet), which turns it into a fully-convolutional segmentation model capable of per-pixel labelling.  Useful for environmental sensing and collision avoidance, segmentation yields dense per-pixel classification of many different potential objects per scene, including scene foregrounds and backgrounds.

The [`segNet`](segNet.h) object accepts as input the 2D image, and outputs a second image with the per-pixel classification mask overlay.  Each pixel of the mask corresponds to the class of object that was classified.

> **note**:  see the DIGITS [semantic segmentation](https://github.com/NVIDIA/DIGITS/tree/master/examples/semantic-segmentation) example for more background info on segmentation.

### Downloading Aerial Drone Dataset

As an example of image segmentation, we'll work with an aerial drone dataset that separates ground terrain from the sky.  The dataset is in First Person View (FPV) to emulate the vantage point of a drone in flight and train a network that functions as an autopilot guided by the terrain that it senses.

To download and extract the dataset, run the following commands from the host PC running the DIGITS server:

``` bash
$ wget --no-check-certificate https://nvidia.box.com/shared/static/ft9cc5yjvrbhkh07wcivu5ji9zola6i1.gz -O NVIDIA-Aerial-Drone-Dataset.tar.gz

HTTP request sent, awaiting response... 200 OK
Length: 7140413391 (6.6G) [application/octet-stream]
Saving to: ‘NVIDIA-Aerial-Drone-Dataset.tar.gz’

NVIDIA-Aerial-Drone-Datase 100%[======================================>]   6.65G  3.33MB/s    in 44m 44s 

2017-04-17 14:11:54 (2.54 MB/s) - ‘NVIDIA-Aerial-Drone-Dataset.tar.gz’ saved [7140413391/7140413391]

$ tar -xzvf NVIDIA-Aerial-Drone-Dataset.tar.gz 
```

The dataset includes various clips captured from flights of drone platforms, but the one we'll be focusing on in this tutorial is under `FPV/SFWA`.  Next we'll create the training database in DIGITS before training the model.

### Importing the Dataset into DIGITS

First, navigate your browser to your DIGITS server instance and choose to create a new `Segmentation Dataset` from the drop-down in the Datasets tab:

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-create-dataset.png" width="250">

In the dataset creation form, specify the following options and paths to the image and label folders under the location where you extracted the aerial dataset:

* Feature image folder:  `NVIDIA-Aerial-Drone-Dataset/FPV/SFWA/720p/images`
* Label image folder:   `NVIDIA-Aerial-Drone-Dataset/FPV/SFWA/720p/labels`
* set `% for validation` to 1%
* Class labels:  `NVIDIA-Aerial-Drone-Dataset/FPV/SFWA/fpv-labels.txt`
* Color map:  From text file
* Feature Encoding:  `None`
* Label Encoding:  `None`

![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-aerial-dataset-options.png)

Name the dataset whatever you choose and click the `Create` button at the bottom of the page to launch the importing job.  Next we'll create the new segmentation model and begin training.

### Generating Pretrained FCN-Alexnet

Fully Convolutional Network (FCN) Alexnet is the network topology that we'll use for segmentation models with DIGITS and TensorRT.  See this [Parallel ForAll](https://devblogs.nvidia.com/parallelforall/image-segmentation-using-digits-5) article about the convolutionalizing process.  A new feature to DIGITS5 was supporting segmentation datasets and training models.  A script is included with the DIGITS semantic segmentation example which converts the Alexnet model into FCN-Alexnet.  This base model is then used as a pre-trained starting point for training future FCN-Alexnet segmentation models on custom datasets.

To generate the pre-trained FCN-Alexnet model, open a terminal, navigate to the DIGITS semantic-segmantation example, and run the `net_surgery` script:

``` bash
$ cd DIGITS/examples/semantic-segmentation
$ ./net_surgery.py
Downloading files (this might take a few minutes)...
Downloading https://raw.githubusercontent.com/BVLC/caffe/rc3/models/bvlc_alexnet/deploy.prototxt...
Downloading http://dl.caffe.berkeleyvision.org/bvlc_alexnet.caffemodel...
Loading Alexnet model...
...
Saving FCN-Alexnet model to fcn_alexnet.caffemodel
```

### Training FCN-Alexnet with DIGITS

When the previous data import job is complete, return to the DIGITS home screen.  Select the `Models` tab and choose to create a new `Segmentation Model` from the drop-down:

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-create-model.png" width="250">

In the model creation form, select the dataset you previously created.  Set `Subtract Mean` to None and the `Base Learning Rate` to `0.0001`.  To set the network topology in DIGITS, select the `Custom Network` tab and make sure the `Caffe` sub-tab is selected.  Copy/paste the **[FCN-Alexnet prototxt](https://raw.githubusercontent.com/NVIDIA/DIGITS/master/examples/semantic-segmentation/fcn_alexnet.prototxt)** into the text box.  Finally, set the `Pretrained Model` to the output that the `net_surgery` generated above:  `DIGITS/examples/semantic-segmentation/fcn_alexnet.caffemodel`

![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-aerial-model-options.png)

Give your aerial model a name and click the `Create` button at the bottom of the page to start the training job.  After about 5 epochs, the `Accuracy` plot (in orange) should ramp up and the model becomes usable:

![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-aerial-model-converge.png)

At this point, we can try testing our new model's inference on some example images in DIGITS.

### Testing Inference Model in DIGITS

Before transfering the trained model to Jetson, let's test it first in DIGITS.  On the same page as previous plot, scroll down under the `Trained Models` section.  Set the `Visualization Model` to Image Segmentation and under `Test a Single Image`, select an image to try (for example `/NVIDIA-Aerial-Drone-Dataset/FPV/SFWA/720p/images/0428.png`):

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-aerial-visualization-options.png" width="350">

Press `Test One` and you should see a display similar to:

![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-digits-aerial-infer.png)

Next, download and extract the trained model snapshot to Jetson.

### FCN-Alexnet Patches for TensorRT

There exist a couple non-essential layers included in the original FCN-Alexnet which aren't supported in TensorRT and should be deleted from the `deploy.prototxt` included in the snapshot. 

At the end of `deploy.prototxt`, delete the deconv and crop layers:

```
layer {
  name: "upscore"
  type: "Deconvolution"
  bottom: "score_fr"
  top: "upscore"
  param {
    lr_mult: 0.0
  }
  convolution_param {
    num_output: 21
    bias_term: false
    kernel_size: 63
    group: 21
    stride: 32
    weight_filler {
      type: "bilinear"
    }
  }
}
layer {
  name: "score"
  type: "Crop"
  bottom: "upscore"
  bottom: "data"
  top: "score"
  crop_param {
    axis: 2
    offset: 18
  }
}
```

And on line 24 of `deploy.prototxt`, change `pad: 100` to `pad: 0`.  Finally copy the `fpv-labels.txt` and `fpv-deploy-colors.txt` from the aerial dataset to your model snapshot folder on Jetson.  Your FCN-Alexnet model snapshot is now compatible with TensorRT.  Now we can run it on Jetson and perform inference on images.

### Running Segmentation Models on Jetson

To test a custom segmentation network model snapshot on the Jetson, use the command line interface to test the segnet-console program.

First, for convienience, set the path to your extracted snapshot into a `$NET` variable:

``` bash
$ NET=20170421-122956-f7c0_epoch_5.0

$ ./segnet-console drone_0428.png output_0428.png \
--prototxt=$NET/deploy.prototxt \
--model=$NET/snapshot_iter_22610.caffemodel \
--labels=$NET/fpv-labels.txt \
--colors=$NET/fpv-deploy-colors.txt \
--input_blob=data \ 
--output_blob=score_fr
```

This runs the specified segmentation model on a test image downloaded with the repo.

![Alt text](https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/segmentation-aerial-tensorRT.png)

In addition to the pre-trained aerial model from this tutorial, the repo also includes pre-trained models on other segmentation datasets, including **[Cityscapes](https://www.cityscapes-dataset.com/)**, **[SYNTHIA](http://synthia-dataset.net/)**, and **[Pascal-VOC](http://host.robots.ox.ac.uk/pascal/VOC/)**.


## Extra Resources

In this area, links and resources for deep learning developers are listed:

* [Appendix](docs/aux-contents.md)
	* [NVIDIA Deep Learning Institute](https://developer.nvidia.com/deep-learning-institute) — [Introductory QwikLabs](https://developer.nvidia.com/deep-learning-courses)
     * [Building nvcaffe](docs/building-nvcaffe.md)
	* [Other Examples](docs/other-examples.md)
	* [ros_deep_learning](http://www.github.com/dusty-nv/ros_deep_learning) - TensorRT inference ROS nodes

