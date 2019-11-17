# **Finding Lane Lines on the Road** 
<font size="2">Alejandra Vargas - 10.10.17</font>

### Project 1


**The goals / steps of this project are the following:**
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---


<div style="text-align: justify"> In this project, I used the tools I learned about in the lesson to identify lane lines on the road.  I developed my pipeline on a series of individual images, and later applied the result to a video stream. </div>

<div style="text-align: justify"> The goal was to piece together a pipeline to detect the line segments in the image and draw them onto the image for display. I tried to average and/or extrapolate the line segments I had detected to map out the full extent of the lane lines.  Ultimately, I draw just one line for the left side of the lane, and one for the right. </div>

<div style="text-align: justify"> The tools I used were: color selection, region of interest selection, grayscaling, Gaussian smoothing, Canny Edge Detection, Hough Tranform line detection and linnear regression. </div>


---


### Reflection


**1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.**

My pipeline consisted of 6 steps. First, I applied GaussianBlur, then I converted the image to HSV space because that makes color segmentation easier, I segmented the white and the yellow. Then the Canny edge detection was applied, I got the desired region of the image applying a mask. The lines where obtained with HoughLines. To finish, the solid color lines where applied to the test images and videos.

In order to draw a single line on the left and right lines, I modified the draw_lines() function by removing all lines with angle between -25 and 25 degress (lines close to horizontal). Then seperated the points that belonged to the left and right line by computing line slope. Finally, obtaining left and right solid lines with linear regression.

In order to get two stable lines, I had to found the average for the slope and y-intercept values in the buffers, fix the y coordinate of the top points and then use the averaged slope and y-intercept to locate the two end points of a line.

To regress a line from x, y and add it to an image I used a linear regressor to fit the data (x,y), removed the outlier, and then fit the cleaned data to get slope and y-intercept in order to find the two end points of the desired line by using slope and y-intercept.


**2. Identify potential shortcomings with your current pipeline**


<div style="text-align: justify">One potential shortcoming would be that for the lane lines detection the color must be stable and for example, in the "challenge.mp4" the algorithm starts to fail when the car drives into the shadow of the trees and the case edge detector tends to grab the (darkest shades) boundaries of the shadow instead of the yellow line under the shadow. The space was modified from RGB to HSVto improve the line extraction.

\
**3. Suggest possible improvements to your pipeline**


<div style="text-align: justify">A possible improvement would be to be able to use curve lines because when the road comes to a sharp curve, the line stops overlapping the lane lines. </div>
.
<div style="text-align: justify">Another potential improvement could be to be able to detect the lines on all the lanes of the road. And be able to tell between wich lines the car is driving. </div>