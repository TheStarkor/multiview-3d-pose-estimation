End-to-end Recovery of Human Shape and Pose
=======
Abstract
----
+ End-to-end framework for reconstructing a full 3D mesh of a human body from a single RGB image.
+ Produce mesh representation that is parameterized by shape and 3D joint angles
+ Minimize reprojection loss of keypoints
  + Can be trained using in-the-wild images that only have GT 2D annotation
+ Introducing adversary trained
  + Tell whether human body shape and pose parameters are real or not
+ Can be trained without any paired 2D-to-3D annotation
  + Infer 3D pose and shape parameters directly form image pixels.

---------------------------

# Intro
## Changes
### Existing work
+ Recovering with 3D joint locaiton alone
  + Don't constrain full degree of freedom
+ Multi-stage approach
  + Not optimal

### HMR
+ Output relative 3D rotation matrices for each joint
+ Implicitly learn joint angle limit
+ End-to-end solution to learn a mapping from image pixels directly to model parameters

## Challenges
### Lack of GT 3D annotation
+ Existing dataset with 3D annotations are captured in constraint environments
### Inherent ambiguities in single-view 2D-to-3D mapping
+ Depth ambiguity
+ Scale ambiguity
### Regressing to rotation matrices
+ Differentiating angle probabilities with reprojection loss is non-trivial and discretiazation sacrifices precision
### Solution
+ There are large-scale 2D key keypoint annotations of in-the-wild images and seperate large-scale dataset of 3D meshs.
+ Take advantage of unpaired 2D keypiont annotations and 3D scans in a conditional generative adversarial manner
  1. Given image, network has to infer the 3D mesh parameters and the camera such that 3D keypoint match the annotated 2D keypoints after projection
  2. To deal ambiguity, these parameters are sent to a discriminator network, whose task is to determine if the 3D parameters correspond to bodies of real human or not
+ Directly regress rotation matrices in an iterative manner

## Beyond existing work
+ Infer 3D mesh parameters  directly from image feature
  + Avoid throwing valuable information
+ Output meshes
+ Trained in an end-to-end manner
+ Result with and _without_ paired 2D-to-3D data

## Evaluate on standard 3D joint location estimation task
-------------
# Related work
## 3D pose estimation
+ Two stage method
  1. Predict 2D joint locations using 2D pose detectors or GT 2D pose
  2. Predict 3D joint locations from 2D joints either by regression or model fitting
  + In order to constrain the inherent ambiguity 2D-to-3D estimation, use various priors   
    ex) limb-length, proportions.
  + Have benefit of being more robust to domain shift   
    But rely on 2D joint detextions and may throw away image information in estimating 3D pose
+ Direct method
  + Dominant approach is fully-convolutional
  + Estimate depth relative to root
  + Use predefined global scale based the average length of bones
  + Images with accurate GT 3D annotations are captured in controlled environment
    + Don't generalize well to the real world
## Weakly supervised 3D
+ Domain gap between MoCap and in-the-wild images
+ In previous work,   
  gain weak supervision from a geometric constraint that encourages relative bone lengths to stay constant.
+ In this work,   
  output 3D joint angles and 3D shape which subsumes these constraints that the limbs should be symmetric
## Methods that output more than 3D joints
+ Methods that fit a parametric body model to manually extracted silhouettes and a few manually provided correspondences.
+ SMPLify   
  optimization-based method to recover SMPL parameters from 14 detexted 2D joints.
  + Not real-time, requiring 20 ~ 60 seconds per image
  + In this work,   
    directly infers SMPL parameters from images instead of detected 2D keypoints and runs in real time
+ VNect   
  Fit rigged skeleton model over time to estimated 2D and 3D joint location
  + Can recover 3D rotations of each joint after optimization
  + In this work,   
    directly output rotations from images
+ [Varol et al.](https://arxiv.org/abs/1701.01370)   
  Use synthetic dataset of rendered SMPL bodies to learn a fully convolutional model for depth and body part segmentation
  DenseReg   
  outputs a dense correspondence map for human bodies
  + In this work,   
    recover all SMPL parameters and the camera from which all of these outputs can be obtained
+ [Kulkarni et al.](https://ieeexplore.ieee.org/document/7299068)   
  Use generative model of body shape and pose with a probabilistic programming framework to estimate body pose from single image   
  [Ten et al.](http://www.bmva.org/bmvc/2017/papers/paper015/index.html)   
  Infer SMPL parameters by first learning a silhouette decoder of SMPL parameters using synthetic data,   
  and then learning an image encoder with the decoder fixed to minimize the silhouette reprojection loss   
  [Tung et al.](https://arxiv.org/abs/1712.01337)   
  Predict SMPL parameters from an image and a set of 2D joint heatmaps   
  + In this work,   
    can be trained without any paired supervision, does not require 2D joint heatmaps as an input and test on images without fine-tuning.
-----
# Model
![image](https://user-images.githubusercontent.com/54238662/94761377-d15c2780-03df-11eb-8516-f9ec8904697a.png)
--------
# Experimental Results
+ No GT mesh 3D annotations exist
+ Evaluate quantitatively on the standard 3D joint estimation task
+ Evaluate auxiliary task of body part segmentation
![image](https://user-images.githubusercontent.com/54238662/94761590-695a1100-03e0-11eb-94fa-2d7e889c21f9.png)
![image](https://user-images.githubusercontent.com/54238662/94761616-75de6980-03e0-11eb-8a1d-e140e101f50a.png)
## 3D joint location estimation
+ Use MPJPE(Mean Per Joint Position Error) and Reconstruction Error metric for Human3.6M dataset
![image](https://user-images.githubusercontent.com/54238662/94761745-c3f36d00-03e0-11eb-9d06-5e596ed0e121.png)
+ In addition to MPJPE, report the PCK(Percentage of Correct Keypoints) thresholded at 150mm and the AUC(Area Under the Curve) over a range of PCK thresholds for MPI-INF-3DHP dataset   
![image](https://user-images.githubusercontent.com/54238662/94761961-67448200-03e1-11eb-9fe0-437410ea127f.png)
## Human body segmentation
+ On the 1000 test images of LSP   
![image](https://user-images.githubusercontent.com/54238662/94762075-b1c5fe80-03e1-11eb-9e25-2e8b5527a1bd.png)