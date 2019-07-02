## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  


The Project
---

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

[image1]: ./test_images/straight_lines2.png "Original Test Image"
[image2]: ./output_images/straight_lines2_undistorted.png "Undistorted Image"
[image3]: ./output_images/straight_lines2_binary.png "Combined Binary Image"
[image4]: ./output_images/straight_lines2_warped.png "Warped Image"
[image5]: ./output_images/straight_lines2_slides.png "Warped Image with Slides"
[image6]: ./output_images/straight_lines2_preReWarp.png "Warped Image with Lane Markings"
[image7]: ./output_images/straight_lines2_reWarp.png "ReWarped Image"
[image8]: ./output_images/straight_lines2_final.png "Final Image"
[video1]: ./test_videos_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

I wanted to try my hand at a markdown this time.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I seperated each goal / step of the project into its own code cell - at least that was the intent - to make things easier to work on.

The code for camera calibration is in the third code cell of the IPython notebook located in "./P2.ipynb".  

I prepared object points which would be appended if I found corners of an image. I assumes the image was on the z plane so that each object point would update along that plane.

I then used the `cv2.calibrateCamera()` function using the resultant `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients.

I created a seperate `undistort_camera` function so that I could view the result. When applying this distortion correction to the test image using the `cv2.undistort()` function I went from this: 

![Original Image][image1]

to this so that I could... 
#### 1. Provide an example of a distortion-corrected image:

![Undistorted Image][image2]

### Pipeline (single images)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 1 through 35 in of the fifth code cell).  Here's an example of my output for this step.

![Combined Binary Image][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `birds_eye()`, which appears in lines 37 through 56 in the fifth code cell of the notebook.  The `birds_eye()` function takes as inputs the original image (`img`) and the combined binary image achieved previously, as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32([[713, 460],
        [1095, 695], 
        [195,695], 
        [570,460]])  
dst = np.float32([[1080, 0],
        [1080, 720], 
        [200, 720], 
        [200, 0]]) 
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Warped Image][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I took the sliding windows approach to confirm I was taking the correct pixels as lane lines (in the fifth code cell, line 58 to line 140). As in the course work, I took 9 windows along the y axis, using a histogram to define where lane edges are. The I appended window indices to achieve an output where left lanes pixels are shown in red and right lane pixels are shown in blue after fitting them to a second order polynomial (line 143 to line 171 of the fifth code cell), using a margin specified by the window indices:

![Warped Image with Slides][image5]

To go further in line 173 to 239 of the fifth code cell, I fit a polynomial to the lane itself and prepared for finding the radius of curvature by finding a "center lane" fit (`center_fitx`). 

![Warped Image with Lane Markings][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the sixth code cell, I used a funtion `measure_curvature()` to transform the `center_fitx` into real world units to find the radius of curvature (lines 1 to 22) and relative position to center (lines 24 to 26).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used the function `bird2original()` to transform the birds eye view of parallel lines back to the original source points in the seventh code cell, lines 1 to 20. This resulted in below:

![ReWarped Image][image7]

Finally, I combined the original image with the reWarped image using the function `cv2.addWeighted()` and added the curvature and distance from center text to final image using the function `cv2.putText()`. This resulted in the final image:

![Final Image][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The largest obstacle, and one I still have yet to figure out, is how to interrupt the curvature update if the lane pixels fit within the window of margin I see a few examples of how it can be done, but am not sure how to fit it into code I've already written or encapsulate a series of functions with it. Without this interrupt, my pipeline takes a lot to compute the resultant videos.

