## Project: Search and Sample Return
---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./images/rover_image.png
[image2]: ./images/example_grid.png
[image3]: ./images/nav_thresh.png
[image4]: ./images/rock_thresh.png
[image5]: ./images/thresh_warp.png
[image6]: ./images/rover.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Please note that the code that was used for this project relies heavily on the tutorials from the website and from the youtube project walkthrough video.

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
The goal is to use the camera data from the rover camera to detect where the rover can drive, identify obstacles and rock samples and ultimately to provide steering and throttle information for the rover. An image from the rover camera can be seen in the following figure:

![alt text][image1]

In the first step the image from the rover camera shall be transformed into a top down view. To identify the perspective of the camera an image that has a reference grid imprinted is used. The edges of the rectangle are measured from the image and now the goal is to transform this image to a warped image where the measuered grid element is in the bottom center position and has a size of ten pixels. To compute the transformation matrix the opencv function "getPerspectiveTransform" is used; the transform is finally implemented with the function "warpPerspective" also from opencv. The rover image with the grid overlay and the resulting warped image can be seen in the following:

![alt text][image2]

In order to separate navigable terrain, obstacles and rocks some color thresholding is applied to the warped image. To identify the navigable terrain the following thresholds are applied to the RGB-color channels of the image:

* Red > 160 AND Green > 160 AND Blue > 160

The channel thresholds are selected as obstacles appear with dark colors while navigable terrain is rather bright. The effect of the threshold can be seen in the following figure:

![alt text][image3]

The original (warped) rover image is shown in the left part of the figure and the navigable terrain that was identfied from the image can be seen in the right part of the figure.

Rocks are identified with the following thresholds:

* Red > 110 AND Green > 110 AND Blue < 50

The thresholds for rock filtering are taken from the project walkthrough video. They are sensible as the rocks are of yellow (gold) color which has a RGB code of (255, 215, 0). An image of a rock and the corresponding filtered image with the rock identified are shown in the following:

![alt text][image4]

Obstacles are defined as the opposite of navigable terrain. If the thresholding is applied to the warped image one has to take care to evaluate only the part of the warped image that actually contains data from the original image. This is done by multiplying the warped image with a previously defined mask that filters out the region that contains no original image data. The mask is generated by applying the transfrom to an image of all ones and the size of the original image.

As the rover position is in the bottom centre of the warped image, in the following a transformation into the rover coordinate system is applied (with x-axis pointing in front of the rover and y-axis pointing to the left). This is done by the function "rover_coords()".

The steering angle for the vehicle can be obtained by computing for each pixel of the navigable terrain the angle to the origin and then by computing the mean over all angles. This turns the vehicle in the direction with the most navigable pixels. Longitudinal motion of the rover can be controlled by evaluating the navigable terrain as well. If too few pixels are detected in front of the rover, the rover can be decelerated and turned around until enough pixels appear in front of the camera again.

The navigable terrain and the navigable terrain warped into rover coordinates (together with the current steering angle) can be seen in the lower part of the following image:

![alt text][image5]

For display purposes the terrain information can be transformed in the world coordinate system as well. This is done with the function "pix_to_world" where the actual warped camera image (in rover coordinates) is rotated into the world coordinate system (function "rotate_pix" with the current rover yaw angle as transformation angle) and then shifted to the actual rover position (function "translate_pix" with current rover position as input). The terrain information is sent to the woldmap list and displayed in red (obstacles), blue (navigable) and white (rocks).

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.
In the following the function "process_image()" from the jupyter notebook is filled with the functions described above. This means that the following steps are performed:

1. apply perspective transform
2. filter warped image for navigable terrain, obstacles and rocks
3. transform filtered image to rover coordinates
4. if required for rover control: compute steering angle
5. transform from rover coordinates to world map
6. diplay obstacles in red channel of world map, navigable terrain in blue channel and obstacles in white (or green channel for perception.py)

A video of the result can be seen in the output folder of the github repository.

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
As already done with the "process_image()" function of the jupyter notebook also the perception.py file is filled with the functions described above. No further modifications were performed. The decision_step() function is used as is.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

I'm running the simulator with a screen resolution of 1024x768 in good grahics quality. The rover is able to navigate around and maps the terrain with the required accuracy - and also detects some rocks :-) (see screenshot below)

![alt text][image6]

But there is certainly lots of room for improvement. Aside from the function to pick up the detected rocks the most potential for improvement in my opinion is in the path planning - where the rover could store the information over already mapped terrain and preferrably drive in the direction of pixels that are detected as navigable for the first time. It might also be interesting to improve the detection part of the project by investigating e.g. if different color spaces (like HSV) can might the detection more robust e.g. in different lighting conditions.