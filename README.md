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

[image1]: ./output_images/test_undst.jpg "Undistorted"
[image2]: ./output_images/undistorted.png "Road Transformed"
[image3]: ./output_images/thresholding.png "Binary Example"
[image_cut]: ./output_images/cropping.png "Cropped Example"
[image4]: ./output_images/perspectiveTransform.png "Warp Example"
[image5]: ./output_images/detectedLines.png "Fit Visual"
[image6]: ./output_images/output.png "Output"
[video1]: ./project_video_edit.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! Note that below I will refer to the different code cells in my IPython notebook where I have made each part separately. The pipeline was later on summarized as functions, which can be seen in code cells 17-18 and 21-25. If you only want to run the pipeline these cells and the 1st cell needs to be run (note that this assumes that you have saved the calibration file).

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the IPython notebook located in "AdvancedLaneLines.ipynb".

In the second code cell I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

In the third code cell I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to one of the calibration images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
This step can be seen in code cell 4 of my notebook. In this step, I applied the distortion correction (saved from camera calibration) to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
In code cell 5 and 6 I used a combination of color and gradient thresholds to generate a binary image. In my final approach I used the pixels which were both present in sobel x and y for the S channel of a HLS image combined with the pixels which were both present in sobel x and y for the R channel in a RGB image. Here's an example of my output for this step, containing other methods as well. What I found was that these were the thresholdings which worked for most of the images, but I could probably experiment more with the values.  (note: this is not actually from one of the test images)

![alt text][image3]

Additionally I was cropping the image to filter away parts that will not be lane markings. This can be seen in code cell 7 and 8 in my notebook. Example of the result:

![alt text][image_cut]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In the 9th code cell of my IPython notebook I used the the thresholded and cropped image and computed the perspective transform using the `cv2.getPerspectiveTransform()` function. I hardcoded the source (`src`) by defining them on the lane markings in the undistorted straight lines images and the destination (`dst`) points by centering around the middle of the image. The following were used:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 215, 710      | 340, 720      | 
| 595, 450      | 320, 0        |
| 690, 450      | 940, 0        |
| 1100, 710     | 940, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial.

By using a histogram of the sum of column pixel values I was able to find the starting points of the lane lines, as seen in code cell 10 of my notebook. Thereafter I used a sliding window to find the rest of the lane markings. A second order polynomial were then fitted to all the found lane marking points (one for left and another for right).

If I had already found the lane markings equations in an earlier image I used this knowledge to not need to search through the whole image again. Instead I searched close to the earlier function predictions, as seen in code cell 12 (This can be seen better in the final pipeline in cell 18 and 23). Here is an example of the output:
![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The vehicle position was calculated based on the offset of the lane markings to the middle of the camera image (we assume that the camera is placed in the middle of the vehicle). This can be seen in code cell 14 of my notebook.

The curvature was calculated for each lane marking (left and right) based on an equation which is explained [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php). This can be seen in code cell 15 of my IPython notebook.

Conversion from pixels to meters was also made for both cases, which is defined based on pixel values in the warped (transformed) image and the assumption that the lane width is 3,7m and that we are predicting 30m ahead.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 16. Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

I think the most important part of the project is to find a good thresholding function. I plotted some different thresholdings and tried to see which thresholdings could capture different parts of the lane markings most correctly in different images and thereafter chose to combine them to get a robust system which can detect lane markings in a lot of different environments. 

I still think that my solution is likely to fail when there is big contrast differences on the road. To make it more robust I will need to analyse for example some image frames of the challenge_video as well as from the harder_challenge_video. I think that if I maybe use a bigger sobel kernel size I could blur the images and get a better result in the end. Also a different combination of thresholding functions may be more appropriate.

Another thing I would like to improve is to use the data from previous video frames better. One idea is to make sure that our new polynomial fit is reasonably close to our latest one. Another needed implementation is to add a new histogram search if the lane markings are lost.

During perspective transform I made the assumption that the lane is flat, but for example in the harder_challenge video it is not, why I would like to investigate how this may affect the system. In that video we can also see that some curves are S-shaped, why a higher order polynomial may be more appropriate.

To see the drawback of my solution it is possible to watch my algorithm in action on [the challenge](./challenge_video_edit.mp4) and [harder_challenge](./harder_challenge_video_edit.mp4) video. 
