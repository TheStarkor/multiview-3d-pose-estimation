# multiview-3d-pose-estimation
Software Projects for Industrial Collaboration(KAIST CS409) with NCSOFT

### Members
KAIST 15 Junkyu Park  [@junQ](https://github.com/jade227)  
KAIST 17 Seungho Baek [@TheStarkor](https://github.com/TheStarkor)  
NCSOFT Vision AI Lab Human Pose Team Yeongeon Lee

## Slides
1. [Proposal Presentation](https://github.com/TheStarkor/multiview-3d-pose-estimation/blob/main/slides/proposal_presentation_team1.pdf)
2. [Mid Presentation](https://github.com/TheStarkor/multiview-3d-pose-estimation/blob/main/slides/mid_presentation_team1.pdf)
3. [Final Presentation](https://github.com/TheStarkor/multiview-3d-pose-estimation/blob/main/slides/final_presentation_team1.pdf)

## Overview


## Contents
1. [Paper Review (HMR, SMPLify, SPIN)]()
   + [HMR](https://github.com/TheStarkor/multiview-3d-pose-estimation/blob/main/paper%20review/HMR.md)
   + [SMPLify](https://github.com/TheStarkor/multiview-3d-pose-estimation/blob/main/paper%20review/SMPLify.md)
   + [SPIN](https://github.com/TheStarkor/multiview-3d-pose-estimation/blob/main/paper%20review/Learning%20to%20reconstruct%203D%20human%20pose%20and%20shape%20via%20model.pdf)
2. [Environment Setting & run SPIN demo]()
3. [2D & 3D parameters]()
   + 2D parameter (numpy)
      |Name|Shape|Description|
      |:----:|:-----:|:-----------:|
      |imgname|1|image path|
      |center|2|position of image center|
      |scale|1|scale of image|
      |part|24 X 3|2D GT<br>24 body parts X ( position(2) + confidence(1) ) |
      |openpose|25 X 3|Openpose result <br> 24 body parts X ( position(2) + confidence(1) ) |
   + 3D parameter (numpy)
      |Name|Shape|Description|
      |:----:|:-----:|:-----------:|
      |pose|72|global orientation (3) <br> 23 joints with axis angle value|
      |shape|10|mesh parameter|
      |has_smpl|1|Boolean value whether dataset has smpl value or not|
      |S|24 X 4|3D GT <br> 24 joints with axis angle |
   + Input parameter (tensor)
      |Name|Shape|Description|
      |:---:|:---:|:--------:|
      |init_pose|1 X 72|pose parameter|
      |init_betas|1 X 10|shape parameter|
      |init_cam_t|1 X 3|camera parameter|
      |camera_center|1 X 2|0.5 * 224 * torch.ones(1, 2, device=0)|
      |joints| 1 X 49 X 3 | openpose result (1 X 25 X 3)<br>+<br>2D GT (1 X 24 X 3)|
4. [Pre-processing]()
5. [Multi-view constraint debugging]()
6. [Generating Pseudo-GT datasets]()
7. [Training & Analyzing]()

## Project Management

## Results

## Conclusion
