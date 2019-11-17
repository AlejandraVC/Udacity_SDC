# Advanced Lane Finding
<font size="2">Alejandra Vargas - 18.12.17</font>

### Project 4
[//]: # (Image References)
---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/camera_calibration.png "Camera Calibration"
[image2]: ./examples/undistort_output.png "Undistorted"
[image12]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_image.png "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/warped_binary.png "Fit Visual"
[image6]: ./examples/lines.png "Lane Lines"
[image7]: ./output_images/test1_output.jpg "Output"
[image8]: ./examples/histogram.png "Histogram"
[video1]: ./project_video.mp4 "Video"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `"./P4.ipynb"`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


#### 1. Provide an example of a distortion-corrected image.


![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (function `find_edges` under `Color and Gradient`).  
The image is first converted to HLS space, and I used the S channel to filter the image, which makes it more robust to locate yellow and white lanes even under different lighting conditions.
Then i used a sobel filter (along x direction) and a gradient direction filter, to filter out most of the horizontal lines. Finally, to handle the cases where more than two lines are found, 
I doubled the weights in the S channel filter. As a result, the yellow line is be more visible than the edge of the road. 

Here's an example of my output for this step:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_t()` (under `Tracking`). First I chose a 
straight line driving image, and selected 4 source points which form a trapezoid along two lane lines, then I defined another 4 destination points in order to get a perspective transform with `cv2.getPerspectiveTransform` function. 

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After applying calibration, thresholding, and a perspective transform to a road image, we should have a binary image where the lane lines stand out clearly. 
However, we still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.


![alt text][image5]

If the input is single image or first frame of the video, a function named `detector` (under `Tracking` cell) is used. It processes the image as follows
* I first take a histogram along all the columns in the lower half of the image. The two most prominent peaks in this histogram 
will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search 
for the lines.  as a reuslt I will get similar figure as follows:

![alt text][image8]


From that point, I use a sliding window, placed around the line centers, to find and follow the lines. It starts at the bottom, and search up to the top of the frame. The idea is visualized as the following figure. 


![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

#### Radius of curvature
The radius of curvature of the curve at a particular point is defined as the radius of the approximating circle. 

Function `measuring_curvature` (under `Measuring Curvature` cell) is implemented for this purpose. 

#### Vehicle position
We assume the camera is mounted roughly at the center of the front window, and we have computed positions of the left and right lines.
So we could just take the bottom position of the left and right lines, and compare it with the middle of the image frame. Also, we could 
utilize the prior knowledge of the width of between left and right lanes (e.g. 3.7 m in US) 
to estimate the distance in real scale.

A function named `measuring_offset` (under `Measuring Offset` cell) is implemented for this purpose. 


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Finally, I warped the detected lane region back to the original input frame. `output_frame` (under `Tracking` cell)).

![alt text][image7]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In order to be implemented, driver assistance systems should reach very low error rates, as high rates irritate drivers and lead to system rejection. Also, there are various scenarios in which the lines and the road appearance can change dramatically. 

Current implementation is based on a monocular camera with computer vision techniques, it involves hard-code perspective transform and 
many manually tuned parameters for lane-line filtering. This maybe sufficient for good condition road driving, e.g. highway, daytime, clear lane marks, 
but it would probably fail with vairous of hard scenarios. In the future, we could consider two approaches:

* Implement more hardware other than only a monocular camera, e.g. Stereo camera, lidar, radar.
* Adopt machine learning techniques.