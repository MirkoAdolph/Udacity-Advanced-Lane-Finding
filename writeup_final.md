## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[cameracalibration]: ./output_images/camera_calibration1.jpg "Camera Calibration"
[undistortedtestimage]: ./output_images/undistorted_test_image.jpg "Undistorted Test Image"
[hls]: ./output_images/hls_original.jpg "hls Color Space"
[hlsbinary]: ./output_images/hls_binary.jpg "hls binary"
[rgb]: ./output_images/rgb_original.jpg "rgb Color Space"
[rgbbinary]: ./output_images/rgb_binary.jpg "rgb binary"
[yuv]: ./output_images/yuv_original.jpg "yuv Color Space"
[yuvbinary]: ./output_images/yuv_binary.jpg "yuv binary"
[lab]: ./output_images/lab_original.jpg "lab Color Space"
[labbinary]: ./output_images/hls_binary.jpg "lab binary"
[xgradientstraight]: ./output_images/xgradient_straight.jpg "xgradient on straight lines"
[xgradient]: ./output_images/xgradient.jpg "xgradient critical situation"
[magdirstraight]: ./output_images/mag_dir_straight.jpg "mag_dir on straight lines"
[magdir]: ./output_images/mag_dir.jpg "mag_dir critical situation"
[warpedimage]: ./output_images/warped.jpg "warped image"
[slidingwindow]: ./output_images/slidingwindow.jpg "sliding window process"
[resultimage]: ./output_images/result_image.jpg "result image"


[hls]: ./project_video.mp4 "Video"
[hls]: ./project_video.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this section is placed in the notebook in the section Camera Calibration.

For camera calibration we are using a function which is provided by OpenCV. This function (cv2.calibrateCamera) takes 2 important inputs - objectpoints and source points. Objectpoints are the points which are referencing to the realworld (i.e. if the distance between two chess corners are 1 cm, the distance between the objectpoints should be 1 cm as well) and source points are the detected points on your image. To find those we can use cv2.findChessboardCorners. To have a reliable calibration you should gather points from at least 20 pictures.

After completing the calibration you end of with calibration variables which are camera specific, means you don't have to calculate them again (as long as you stick to the same camera). Below you can see the result of an undistorted image:

![alt text][cameracalibration]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

As alread explained above, as soon as you calculated your distortion parameters you can easily correct the images by using cv2.undistort. An example how an undistorted test image looks like can be seen below:
![alt text][undistortedtestimage]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

This was actually the main work of the project. To have a reliable lane detection that provides a decent detection over all test images as well as over the test stream needed a lot of investigation in different methods. Methods can be divided in 2 big categories:

COLOR CHANNELS (you can find the code for all these in the notebook in the section color channels)
For this I was looking into multiple color spaces (HLS, RGB, YUV, LAB), below you can see the results for each color space:

![alt text][hls]
![alt text][hlsbinary]
![alt text][rgb]
![alt text][rgbbinary]
![alt text][yuv]
![alt text][yuvbinary]
![alt text][lab]
![alt text][labbinary]

In the first steps I mainly used the s channel of the HLS color space as well as the combination of y and u channel of the YUV color space. This combination managed to create good results over 80% of the video but as soon as there was shadow or a colorchange of the asphalt these funtions created a lot of noise which lead to a failure of my algorithm. That was the reason why I switched to a combination of the l channel (HLS color space) and the b channel.

The l channel of the HLS color space captured pretty well all white lane markings while the b channel captured the yellow side lane pretty well. Combination of both provided a really good lane detection through the whole project.

GRADIENTS (you can find the code for all these in the notebook in the section gradients)

During the project I also applied sobel to calculate a gradient in x direction as well as a magnitude of x and y gradient and the direction of the combined x y gradient. The last two mentioned I combined to one binary because otherwise the binary would be to noisy. While the x gradient provides us pretty good results for straight lines on dark asphalt, it nearly gives us any input in the critical situation that were mentioned already above.

![alt text][xgradientstraight]
![alt text][xgradient]

Same behaviour as for the x gradient can be seen by inspecting the combination of magnitude and direction of the resulting gradient of x and y.

![alt text][magdirstraight]
![alt text][magdir]

Result: I started using a combination of gradients (xgradient, combination of magnitude and direction) and color channels (s channel, combination of y l channel) which provided good results for 80% of the video. In critcal situations this approach was not feasible. In the end I just used a combination of two color spaces which provided good results over the complete video (l and b channel).

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Code for warping can be found in the notebook in the section Warp Image.

To calculate a transformation matrix to perform a perspectife transform from one picture to another you have to provide 4 points in the original image (also called source points) as well as 4 points in the transformed image, these points are the location from the source points in a new image. To determine those points, I choosed a test image which displays straight lines so that I could test if my warped image displays a straight line as well in the top view. With try and error I ended up with 4 source and destination points that created the following result for the perspective transform.

![alt text][warpedimage]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Code for identifing lane pixels can be found in Final Pipeline Library (since the notebook includes all test runs that I tried please only refer to the Final Pipeline Library at the end of the notebook!).

For the identification of lane pixels and the fitting process I used the code that was provided during our lecture sessions and modified it slightly to fit into my pipeline. Rough Explanation of whats going on in this code snippet is (find_lane_pixels):
- Create a Histogram (take only the bottom half of the picture into account)
- Identify the peaks in the histogram to get a starting point for our left and right lane
- Create multiple windows which are placed vertical over each other
- Iter through each window and filter if pixels are in there and if the number of detected pixels exceed a given treshold, adjust the center of the window
- With this process you end up with points for the left and right lane which can then be forwarded to np.polyfit and achieve the fitting of the line.

![alt text][slidingwindow]

Picture above shows a state of the pipeline where lane detections binary where not improved but still captures the procedure of the sliding window process pretty well.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Calculation can be found in Final Pipeline library in def measure_curvatur_pixels and in def fit_polynomial (for center offset, this was not moved in another function since we already have the needed data available in this function - could be improved by outsourcing in a one function).

The curvature of the radius was implemented the same way as we learned it in our lesson. Important was that we had to transform from pixel space to real world space to get a result in meters. Therefore we had to calculate two parameters - ym_per_pix and xm_per_pix.

xm_per_pix could be calculated easily by our warped test image. There we could clearly see that the lane width was captured by 380 pixels. According to our lessons a lane in the USA is approximately 3.7 m width, which results in a xm_per_pixel of 3.7m/380Pixel.

ym_per_pix was calculated by making the assumption that the length of the captured street (masked with the region of interest) is approximately 30 meters which resulted in a ym_per_pixel of 30m/720Pixel.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This part is implemented in Final Pipeline Library in def plot_result. An example of how this looks like is given below:

![alt text][resultimage]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As already pointed out in the steps before the most challenging part was to find a method that creates a binary image which consistently provides good results over multiple critical situations. The first approach was to go for gradients, the s-channel (HLS space) and a combined y-u channel (YUV space). Problem was here that as soon as we entered a gray asphalt section that we couldn't detect any lanes over our gradient methods and the s-channel was adding some more noise to the final picture. These were the reason for the fail of this approach.

After this I investigated the combination of l-channel (HLS space) and b-channel (LAB space) since I recognized that the l-channel detects pretty well white lines while the b-channel detects pretty well the yellow lines. After moving to this approach the pipeline instantly created good results and I was able to detect the lane through the complete video. Since this approach already generated good results I did not investigate further in the direction of tracking and performance improvement due to time issues.

IMPROVEMENT

Currently for each image we calculate the lines via the sliding window approach. To optimize our performance we should switch to a faster solution for calculating the following lines, as soon as we got a first pair of detected lane lines. Approach here would be to search in a customized area around the already detected lines for new lines.

We are not tracking our lines which could higly improve our algorithm in robustness. For example checking if the last curvature is close to the new calculated curvature could already be an indicator if this line is feasible or not.

Since we are not tracking we also cant fall back to a previous solution, that means as soon as we detect no lane line our algorithm won't create results which will result in a failure of the pipeline. That is crictical and has to be adapted.

Improve test coverage. Currently we only run our algorithm at specific test images and videos. In the real world there are many more critical situations that can appear and have to be tested if the algorithm is robust enough to still detect the lane lines.
