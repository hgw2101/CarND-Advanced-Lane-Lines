##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./write_up_images/image1.png "Undistorted Chessboard"
[image2]: ./write_up_images/image2.png "Undistorted Car Image"
[image3]: ./write_up_images/image3.png "Binary Example"
[image4]: ./write_up_images/image4.png "Warp Example"
[image5]: ./write_up_images/image5.png "Sliding Window"
[image6]: ./write_up_images/image6.png "Lane Polynomial"
[image7]: ./write_up_images/image7.png "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for camera calibration is in the 2nd, 3rd and 4th code cell of the `model.ipynb` notebook. I calculated the camera matrix and distortion coefficients by first collecting a group of 'object points', which are the actual coordinates in 3D space (x, y, z) of a chessboard. The x and y coordinates will vary depending on the position of each chessboard point, while the z coordinate will always be 0, since we assume the chessboard is on a flat surface. Then I collected the 'image points' of the chessboard, which represent the actual coordinates of the chessboard points in the calibration image, using OpenCV's `findChessboardCorners` method. With both object points and image points, I was able to calculate the camera matrix, distortion coefficients using OpenCV's `calibrateCamera` and the undistorted image using the `undistort` method.

Here is an example of a distorted, i.e. raw, image and an undistorted image:

![undistort][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
Here is an example of a test image in its original form versus what it looks like after correcting for camera distortion:
![undistort test][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color, using the HLS color space, and absolute gradient in the x direction. I chose color transformation in the HLS color space and in particular, the S/Saturation channel because it gave the best result in clearly identifying lane lines. I chose the absolute gradient in the x direction because it appears to be more robust than other gradient techniques: the absolute gradient in both x and y directions seems to pick up a lot of noise, while the direction of the gradient transform seems completely unhelpful.

I then combined the color and gradient transform into a single method, `transform_pipeline` so that only pixels that satisfy both the color and gradient requirements will be recorded as `1` in the binary output. The code for this method can be found in the 12th code cell.

Here is an example of what a original image looks like after the color and gradient transformation:

![color gradient transformation][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper`, which is in the 13th code cell in the `model.ipynb` notebook. The `warper` takes the following 5 parameters/arguments: 
`img`: the image that we want to perform perspective transformation on
`camera_matrix`: the camera matrix that we calculated using the camera calibration logic above
`distortion_coefficients`: the distortion coefficients we calculated using the camera calibration logic above
`source`: the source points in the original image
`destination`: the destination points of where we want the source points to be in the transformed image

I arbitrarily picked 4 source points based by locating the bottom-most and top-most points on the left and right lane lines, and the corresponding destination points are basically 4 points on a square:

```python
src = np.float32([[275,669],[580,461],[703,461],[1025,669]])
dst = np.float32([[275,720],[280,100],[1020,100],[1025,720]])
```

I verified that my perspective transform was working as expected by drawing the `source` and `destination` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![perspective transform][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This took me the longest :). I initially identified which pixel points belong to the left lane line and which ones for the right lane line using a sliding window technique, using the `fit_sliding_window` method in the 16th code cell. Once I have identified these respective points, I was able to use `numpy`'s `polyfit` function to plot a second order polynomial to approximate the lane line. One I have fit one image frame with a polynomial, I can identify left and right lane line pixels in the subsequent images by searching for pixels within a certain margin of the lane line polynomial in the previous image. This is done using the `find_lane_lines` method in the 19th code cell.

Here is an example of a polynomial using the `fit_sliding_window` method:

![sliding window][image5]

Here is an example of a polynomial using the `find_lane_lines` method:

![poly margin][image6]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I defined a method called `lane_curvature` in the 21st code cell in the `model.ipynb` notebook. I made sure that this method returns the curvature in meters, not pixels.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the overall pipeline, `pipeline`, for processing each image in a test image. This is in the 25th code cell in the `model.ipynb` notebook. Here is an example of my result on a test image:

![lane overlay][image7]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had a lot of problems while implementing the sliding window technique to identify pixels that belong to the left versus right lane lines, mostly because I was unfamiliar with the peaks in a histogram approach and that the binary images with window frames were not displaying correctly in the `matplotlib.pyplot` API because the image pixel values ranged from 0 to 1 as opposed to 0 to 255. It's where I got stuck at first and decided to move onto project 5, after that, I discovered the mistakes I made and got through the rest of the project.

My implementation seems a bit slow, it only processes about 3 frames per second, which is way too slow in practice. I tried to speed it up by using the `find_lane_lines` method, as opposed to `fit_sliding_window` method, to plot polynomial lines, but did not see any noticeable increase in speed. There are probably redundancies in my code that were copying images that were not being used in the final result and they can be removed.
