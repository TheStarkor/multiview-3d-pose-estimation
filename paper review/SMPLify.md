Keep it SMPL : Automatic Estimation of 3D Human Pose and Shape from a Single Image   
=====
[Link](https://arxiv.org/abs/1607.08128)
# Abstract
+ First method to automatically estimate the 3D pose of the human body as well as its 3D shape from a single unconstrained image
+ Challenging because of the complexity of human body, articulation, occlusion, clothing, lighting and the inherent ambiguity in inferring 3D from 2D
+ Use CNN-based method, DeepCut, to predict(bottom-up) 2D body joint location then fit(top-down) SMPL to 2D joints
  + By minimizing error between projected 3D model joints and detected 2D joints
+ Evaluate on the Leeds Sports, HumanEva, and Human3.6M datasets
# Introduction
+ Fully automatic and estimates a 3D mesh capturing both pose and shape from a 2D image
+ SMPLify
  1. Estimate 2D joints using a recently proposed CNN called DeepCut **(bottom up)**   
    Successful at estimating 2D human pose but not 3D pose and shape from one image
  2. Estimate 3D pose and shape from the 2D joints using 3D generative model called SMPL **(top down)**
+ Exploits high-quality 3D human body model(ex. SMPL) that is trained from thousands of 3D scans and hence captures the statistics of shape variation in the population as well as how people deform with pose 
+ Define objective function and optimize pose and shape directly   
  so that the projected joints of 3D model are close to the 2D joints estimated by the CNN
+ Using generatvie 3D model
  + Enables us to reason about interpenetration
+ Provide an interpenetration term that is differentiable with repect to body shape and pose
+ If we don't know the gender, fit gender-neutral model to images    
  If we know the gender, use gender-specific model for better results
+ To deal with pose ambiguity, we train a prior over pose from SMPL models that have been fit to the CMU mocap marker data using MoSh
-------
# Related work
+ Most methods formulated the problem as finding a 3D *skeleton* such that its 3D joints project to known or estimated 2D joints
  + In this work, we take shape to mean the pose-invariant surface of the human body in 3D and distinguish this from pose which is the articulated posture of the limbs
## 3D pose from 2D joints
+ Methods all assume known correspondence between 2D joints and a 3D skeleton   
  Methods make different assuptions about the statistics of limb-length variation
+ Recent methods typically use the CMU dataset and learn a statistical model of limb lengths and poses from it
+ Above approaches have weak, or non-existent, models of human shape
+ In this work, we argue that a stronger model of body shape, learned from thousands of people, captures the anthropometric constraints of the population   
+ We can model interpenetration, avoiding impossible poses because we have 3D shape
## 3D pose and shape
+ Work on estimating 3D body shape from single images
  + This work assumes good silhouettes are available
+ No previous method estimates 3D body shape and pose directly from only **2D joints**
  + Given good statistical model(SMPL), our approach works surprisingly well
  + SMPL
    + Has explicit 3D joints   
    We fit their projection directly to 2D joints
    + Defines how joint locations are related to the 3D surface of the body   
    Enable inference of shape from joints
    + Doesn't represent anatomical joints   
    It represents them as a function of surface vertices   
    This couples joints and shape during model training and means that solving for them together is important
## Making it automatic
+ Most methods assume known correspondences, and some involve significant manual intervention
+ There are few methods that try to solve the entire problem of inferring 3D pose from a single image
+ Recent advances in deep learning are producing methods for estimating 2D joint positions accurately
  + We use DeepCut method
+ recent work uses a CNN to estimate 2D joint locations and then fit 3D pose to these using a monocular video sequence   
  They don't show result for single images
+ None of these automated methods estimate 3D body shape
  + We demonstrate a complete system that uses 2D joint detections and fits pose and shape to them from a single image
-----------------
# Method 
![image](https://user-images.githubusercontent.com/54238662/94791777-16e61800-0413-11eb-88d0-47eb73a97ecc.png)
1. Take single input image
2. Use DeepCut CNN to predict 2D body joint (J<sub>est</sub>)
3. For each 2D joint *i*, the CNN provides a confidence value, w<sub>i</sub>
4. Fit 3D body model such that the projected joints of the model minimize a robust weighted error term
+ The body model is defined as a function     
  <p align = "center"><image src = "https://user-images.githubusercontent.com/54238662/94888629-b7404900-04b4-11eb-9fda-eda0d3c90fe8.png">   
  <p align = "center"><image src = "https://user-images.githubusercontent.com/54238662/94888649-c8895580-04b4-11eb-9b36-de64f8b33e20.png">   

  
  The output is a triangulated surface with 6890 vertices
+ Shape parameters <image src = "https://user-images.githubusercontent.com/54238662/94889684-063bad80-04b8-11eb-887a-22b3a5efae04.png"> are coefficients of a low-dimensional shape space, learned from a training set of thousands of registered scans
+ The pose of body is defined by a skeleton rig with 23 joints   
  Pose parameters <image src = "https://user-images.githubusercontent.com/54238662/94889711-1eabc800-04b8-11eb-8169-0af48bca84be.png"> represent the axis-angle representation of the relative rotation between parts
+ Let <image src = "https://user-images.githubusercontent.com/54238662/94889746-3420f200-04b8-11eb-9a2a-5e4b9620e77d.png"> be the function that predicts 3D skeleton joint locations from body shape
+ Joints can be put in arbitary poses by applying a global rigid transformation   
  We denote posed 3D joints as <image src = "https://user-images.githubusercontent.com/54238662/94889891-ac87b300-04b8-11eb-8e53-2e82a1e5e90e.png"> for joint *i*, where <iamge src = "https://user-images.githubusercontent.com/54238662/94889928-c628fa80-04b8-11eb-92e4-525f1ea87a07.png"> is the global rigid transformation induced by pose <image src = "https://user-images.githubusercontent.com/54238662/94889711-1eabc800-04b8-11eb-8169-0af48bca84be.png">
+ To project SMPL joints into the image we use a perspective camera model, defined by parameters K
## Approximating bodies with capsules
![image](https://user-images.githubusercontent.com/54238662/94799912-d4c2d380-041e-11eb-8618-3b2a0d2dd21d.png)
+ Advantage of our 3D shape model is that it allows us to detect and prevent interpenetration between body parts
+ We use proxy geometries to compute collisions and approximate the body surface as a set of **capsules**
+ We train regressor from model shape parameters to capsule parameters(axis length, radius), and pose the capsules according to '<image src = "https://user-images.githubusercontent.com/54238662/94889928-c628fa80-04b8-11eb-92e4-525f1ea87a07.png">', the rotation induced by the kinematic chain
1. Start from capsules manually attached to body joints in the template
2. Perform gradient-based optimization of their radii and axis lengths to minimize the bidirectional distance between capsules and body surfaces
3. Learn a linear regressor from body shape coefficients(<image src = "https://user-images.githubusercontent.com/54238662/94889684-063bad80-04b8-11eb-887a-22b3a5efae04.png">) to the capsules'radii and axis lengths using cross-validated ridge regression
4. Once the regressor is trained, the procedure is iterated once more, initializeing the capsules with the regressor output
## Objective function
+ Sum of five error terms   
  A joint-based data term, three pose priors, and a shape prior
  <p align = "center"><image src = "https://user-images.githubusercontent.com/54238662/94889439-2b7bec00-04b7-11eb-8a2f-5097cbc72747.png">   
  <p align = "center"><image src = "https://user-images.githubusercontent.com/54238662/94889449-346cbd80-04b7-11eb-8f4d-b93541930889.png">
**Detailed equation skipped temporary**
## Optimization
+ Assume that camera translation and body orientation are unknown   
+ Require that the camera focal length or its rough estimate is known
+ Initialize camera translation by assuming that the person is standing parallel to the image plane
+ Estimate the depth via the ratio of similar triangles defined by the torso length of the mean SMPL shape and the predicted 2D joints
+ Since this assumption is not always true, we further refine this estimate by minimizing E<sub>J</sub> over the torso joints alone with respect to camera translation and body orientation
+ Keep <image src = "https://user-images.githubusercontent.com/54238662/94889684-063bad80-04b8-11eb-887a-22b3a5efae04.png"> fixed to the mean shape during this orientation
+ Don't optimize focal length since the problem is too unconstrainted to optimize it together with translation
+ After estimating camera tranaslation, we fit our model by minimizing objective function in a staged approach
+ When the subject is captured in a side view, assessing in which direction the body is facing might be ambiguous   
+ To address this, we try two initializations when the 2D distance between the CNN-estimated 2D shoulder joints is below a threshold
  + With body orientation estimated as above
  + With that orientation rotated by 180 degrees
+ Finally pick the fit with lowest E<sub>J</sub>
# Evalutation
+ Evaluate accuracy of both 3D pose and 3D shape estimation
  + 3D pose
    + Quantitative evaluation
      + Use HumanEva-I and Human3.6M (GT datasets that have restricted laboratory environment and limited poses)
    + Qualitative evaluation
      + Use LSP
  + 3D shape
    + Quantitative evaluation
      + There are few images with GT 3D shape
      + Use synthetic data to evaluate how well shape can be recovered form 2D joints corrupted by noise
## Quantitative evaluation : synthetic data
![image](https://user-images.githubusercontent.com/54238662/94833609-4c5a2800-044a-11eb-9cf2-60bddb63b958.png)
## Quantitative evaluation : real data
![image](https://user-images.githubusercontent.com/54238662/94840693-c93dcf80-0453-11eb-8822-08559eac2f41.png)
![image](https://user-images.githubusercontent.com/54238662/94840724-d8248200-0453-11eb-80b7-348484c1fa37.png)
![iamge](https://user-images.githubusercontent.com/54238662/94840752-e1adea00-0453-11eb-8813-59c931df270e.png)
![image](https://user-images.githubusercontent.com/54238662/94840799-f8544100-0453-11eb-84e5-797b14284720.png)
## Qualitative evaluation
![image](https://user-images.githubusercontent.com/54238662/94841203-a2cc6400-0454-11eb-95e6-6c5524a40456.png)
![image](https://user-images.githubusercontent.com/54238662/94841235-aeb82600-0454-11eb-8299-6aea1e2fdee4.png)