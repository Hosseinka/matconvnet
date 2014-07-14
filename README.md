# MatConvNet: CNNs for MATLAB

Version 1.0-beta.

**MatConvNet** is a simple MATLAB toolbox implementing *Convolutional
Neural Networks* (CNNs) for computer vision applications. Its main
features are:

- *Flexibility.* Neural network layers are implemented in a
  straightforward manner, often directly in MATLAB code, so that they
  are easy to modify, extend, or integrate with new ones. Other
  toolboxes hide the neural network layers behind a wall of compiled
  code; here the granularity is much finer.
- *Power.* The implementation can run the latest models such as
  Krizhevsky et al., including the DeCAF and Caffe
  variants. Pre-learned features for different tasks are provided.
  Importing and exporting of models between implementations should be
  a simple affair.
- *Efficiency.* The implementation is quite efficient, supporting both
  CPU and GPU computation. Despite MATLAB overhead, it is only
  marginally slower than alternative implementations.

This library may be merged in the future with
[VLFeat library](http://www.vlfeat.org/). It uses a very similar
style, so if you are familiar with VLFeat, you should be right at home
here.

## Installation

This library comprises several MEX files that need to be compiled
before MATLAB can use it.

### Download

You can download a copy of the source code here:

- [Tarball for version 1.0-beta](download/matconvnet-1.0-beta.tar.gz)
- [GIT repository](http://www.github.com/vlfeat/matconvnet.git)

### Compiling

Compiling the CPU version of MatConvNet is a simple affair. The simple
method is to use supplied `Makefile`:

    > make ARCH=<your arch> MATLABROOT=<path to MATLAB>

This requires MATLAB to be correctly configured with a suitable
compiler (usually XCode for Mac, gcc for Linux, Visual C for Windows).
For example:

    > make ARCH=maci64 MATLABROOT=/Applications/MATLAB_R2014a.app

would work for a Mac with MATLAB R2014 installed in its default
folder. Other supported architectures are `glnxa64` (for Linux) and
`win64` (for Windows).

Compiling the GPU version should be just as simple; however, in
practice, this may require some configuration. First of all, you will
need a recent version of MATLAB (e.g. R2014a). Secondly, you will need
a corresponding version of the CUDA (e.g. CUDA-5.5 for R2014a) -- use
the `gpuDevice` MATLAB command to figure out the proper version of the
CUDA toolkit. Then

    > make ENABLE_GPU=y ARCH=<your arch> MATLABROOT=<path to MATLAB> CUDAROOT=<path to CUDA>

should do the trick. For example:

    > make ENABLE_GPU=y ARCH=maci64 MATLABROOT=/Applications/MATLAB_R2014a.app CUDAROOT=/Developer/NVIDIA/CUDA-5.5

should work on a Mac with MATLAB R2014a.

### Setting up MATLAB

Once the MEX files are properly compiled, MATLAB setup is easy. Simply
start MATLAB and type

    > run <path to MatConvNet>/matlab/vl_setupnn

At this point the library should be ready to use. To test it, try
issuing the command:

    > vl_test_nnlayers

## Getting started

This section provides a practical introduction to this toolbox.
Please see the [reference PDF manual](matconvnet-manual.pdf) for
technical details.

MatConvNet can be used to [train CNN models](#training), or it can be
used to evaluate several [pre-trained models](#pretrained) on your data.

### Using pre-trained models <span id='pretrained'></span>

You can download the following models:

- [ImageNet-S](). A relatively small network pre-trained on
  ImageNet. Similar to Krizhevsky et al. 2012 network.
- [ImageNet-M](). A somewhat larger network, with better performance.
- [ImageNet-L](). A large model.

For example, in order to run ImageNet-S on a test image, use:

    gunzip('http://www.vlfeat.org/matconvnet/models/imagenet-s.mat') ;
    net = load('imagenet-s.mat') ;
    im = imread('peppers') ;
    res = vl_simplenn(net, 'preprocess', im) ;

`vl_simplenn` is a wrapper around MatConvNet core computational
functions that simplifies evaluating CNN with a simple linear topology
(i.e. chains of layers). It is not needed to use the toolbox, but it
simplifies common examples such as the ones discussed here. The
`preprocess` option is required to convert the image `im` into a
format which can be processed by the network; typically, this involves
rescaling/cropping the image to a fixed side, converting it to
`single` precision, and subtracting an image mean. If you want to do
these steps by yourself, you can remove 'preprocess'.

`res` is a structure containing the network evaluation.

    y = res(end).x ;
    [score, class] = max(y) ;
    fprintf('estimated class: %s (score: %f)\n', net.meta.classes{class}, score) ;

### Training your own models <span id='training'></span>

MatConvNet can be used to train models using stochastic gradient
descent and back-propagation.

The following learning demos are provided:

- **MNIST**. See `cnn_mnist`.
- **CIFAR**. See `cnn_cifar`.
- **ImageNet**. See `cnn_imagenet`.

These are self-contained, and include downloading and unpacking of the
data. MNIST and CIFAR are small datasets (by today's standard) and
training is feasible on a CPU. For ImageNet, however, a powerful GPU
is highly recommended.

### Working with GPU acelereated code

GPU support in MatConvNet relies on the corresponding support as
provided by recent releases of MATLAB and of its Parallel Programming
Toolbox. This toolbox relies on CUDA-compatible cards, and you will
need a copy of the CUDA devkit to compile GPU support in MatConvNet
(see above).

All the core computational functions (e.g. `vl_nnconv`) in the toolbox
can work with either MATLAB arrays or MATLAB GPU arrays. Therefore,
switching to use the GPU is then as simple as converting data
contained in arrays (e.g. images) in GPU arrays before calling the
toolbox functions.

In order to make the very best of powerful GPUs, it is important to
balance the load between CPU and GPU in order to avoid starving the
latter. In training on a problem like ImageNet, the CPU(s) in your
system will be busy loading data from disk and streaming it to the GPU
to evaluate the CNN and its derivative.

## Copyright and acknowledgments

This package was created and is currently maintained by
[Andrea Vedaldi](http://www.robots.ox.ac.uk/~vedaldi). It is
distributed under the permissive BSD license (see the file `COPYING`).

    Copyright (c) 2014 Andrea Vedaldi.
    All rights reserved.

    Redistribution and use in source and binary forms are permitted
    provided that the above copyright notice and this paragraph are
    duplicated in all such forms and that any documentation,
    advertising materials, and other materials related to such
    distribution and use acknowledge that the software was developed
    by the <organization>. The name of the <organization> may not be
    used to endorse or promote products derived from this software
    without specific prior written permission.  THIS SOFTWARE IS
    PROVIDED ``AS IS'' AND WITHOUT ANY EXPRESS OR IMPLIED WARRANTIES,
    INCLUDING, WITHOUT LIMITATION, THE IMPLIED WARRANTIES OF
    MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.

The implementation of the fundamental computational blocks in this
library, and in particular of the convolution operators, is inspired
by [Caffe](http://caffe.berkeleyvision.org).

## Changes

- 1.0-beta. First public release (summer 2014).
