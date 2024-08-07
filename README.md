# SFND 3D Object Tracking

Welcome to the final project of the camera course. By completing all the lessons, you now have a solid understanding of keypoint detectors, descriptors, and methods to match them between successive images. Also, you know how to detect objects in an image using the YOLO deep-learning framework. And finally, you know how to associate regions in a camera image with Lidar points in 3D space. Let's take a look at our program schematic to see what we already have accomplished and what's still missing.

<img src="images/course_code_structure.png" width="779" height="414" />

In this final project, you will implement the missing parts in the schematic. To do this, you will complete four major tasks: 
1. First, you will develop a way to match 3D objects over time by using keypoint correspondences. 
2. Second, you will compute the TTC based on Lidar measurements. 
3. You will then proceed to do the same using the camera, which requires to first associate keypoint matches to regions of interest and then to compute the TTC based on those matches. 
4. And lastly, you will conduct various tests with the framework. Your goal is to identify the most suitable detector/descriptor combination for TTC estimation and also to search for problems that can lead to faulty measurements by the camera or Lidar sensor. In the last course of this Nanodegree, you will learn about the Kalman filter, which is a great way to combine the two independent TTC measurements into an improved version which is much more reliable than a single sensor alone can be. But before we think about such things, let us focus on your final project in the camera course. 

## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
  * Install Git LFS before cloning this Repo.
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level project directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./3D_object_tracking`.

## Rubric Fulfillment
- [x] FP.1 Match 3D objects
  * Description: Implement the method "matchBoundingBoxes", which takes as input both the previous and the current data frames and provides as output the ids of the matched regions of interest (i.e. the boxID property). Matches must be the ones with the highest number of keypoint correspondences.
  * Implementation: The **matchBoundingBoxes** is implemented `from Line 251 to 268` in the camFusion_Student.cpp file.
- [x] FP.2 Compute Lidar based TTC
  * Description: Compute the time-to-collision in second for all matched 3D objects using only Lidar measurements from the matched bounding boxes between current and previous frame.
  * Implementation: The **computeTTCLidar** is implmented `from Line 220 to 248` in the camFusion_Student.cpp file.
  * Note: Outliers are removed based on the following criteria:
    1. If they are too far away from the mean value of the x-coordinates of Lidar points (i.e., the ratio of the outlier to the mean value is smaller than 0.8).
    2. If they are not within the ego lane.
- [x] FP.3 Associate Keypoint Correspondences with Bounding Boxes
  * Description: Prepare the TTC computation based on camera measurements by associating keypoint correspondences to the bounding boxes which enclose them. All matches which satisfy this condition must be added to a vector in the respective bounding box.
  * Implementation: The **clusterKptMatchesWithROI** is implemented `from Line 139 to 161` in the camFusion_Student.cpp file.
  * Note: Outliers in the matches were removed if their Euclidean distances were too far from the average distance. Specifically, if the ratio of the outlier's distance to the mean distance was smaller than 0.85, it was considered an outlier and removed.
- [x] FP.4 Compute Camera-based TTC
  * Description: Compute the time-to-collision in second for all matched 3D objects using only keypoint correspondences from the matched bounding boxes between current and previous frame.
  * Implementation: The **computeTTCCamera** is implemented `from Line 165 to 217` in the camFusion_Student.cpp file.
  * Note:  
    1. Outliers in the distance ratios were removed if they are not within the range of first quartile and third quartile.
    2. Lidar TTC was calculated using the mean value of the inliers.
- [x] FP.5 Performance Evaluation 1 (Lidar TTC result)
  * Description: Find examples where the TTC estimate of the Lidar sensor does not seem plausible. Describe your observations and provide a sound argumentation why you think this happened.
  * Lidar TTC Performance Observation:
    In the first three frames, the TTC increases from 12.3 to 16.4 seconds. However, the images show that the ego vehicle is getting closer to the front vehicle, indicating that the relative distance is decreasing. According to the Lidar-based TTC formula, this suggests that the relative velocity between the ego car and the front car is decreasing, causing the TTC to increase even though the relative distance is smaller.
    Additionally, from frame 11 to frame 17, we observe that the TTC in frame 16 suddenly jumps to 11 seconds, despite the fact that everything looks normal in the images during this period.
    Based on these two observations, we can conclude that the constant velocity model doesn't estimate TTC accurately in this scenario. In urban areas, where stop-and-go situations are common, acceleration and deceleration occur frequently. Therefore, using a constant acceleration model is much more suitable for tracking the front vehicle.
- [x] FP.6 Performance Evaluation 2 (Camera TTC result)
  * Description: Run several detector / descriptor combinations and look at the differences in TTC estimation. Find out which methods perform best and also include several examples where camera-based TTC estimation is way off. As with Lidar, describe your observations again and also look into potential reasons.
