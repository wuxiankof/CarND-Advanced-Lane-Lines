# Project 2: Advanced Lane Finding
---

## Project Objectives

The goals / steps of this project are the following:

1. Camera Calibration
    - Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Pipeline (test images)
    1. Apply a distortion correction to raw images.
    2. Use color transforms, gradients, etc., to create a thresholded binary image.
    3. Apply a perspective transform to rectify binary image ("birds-eye view").
    4. Detect lane pixels and fit to find the lane boundary.
    5. Determine the curvature of the lane and vehicle position with respect to center.
    6. Warp the detected lane boundaries back onto the original image.
3. Pipeline (video)
    - Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
4. Discussion

[//]: # (Image References)

[image1]: ./output_images/1_Camera_Calibration.jpg "Undistorted"
[image2]: ./output_images/2_1_distortion_correction.jpg "Distortion Correction"
[image3]: ./output_images/2_2_binary_image.jpg "Binary Example"
[image4]: ./output_images/2_3_Warp_Perspective.jpg "Warp Example"
[image5]: ./output_images/2_4_fit_Polynomial.jpg "Fit Visual"
[image6]: ./output_images/2_5_rapred_back.jpg "Output"
[video1]: https://youtu.be/9eNAkmtv1G8 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

- Frist of all, I have defined a function `Read_Test_Images(...)` to read all chessboard images from the "test_images" folder, then identified $n_x \times n_y$ points and returned their indices and coordinates (in pixels) through two variables `objpints` and `imgpoints`, respectively.
- Next, I have obtained the calibration parameters, i.e., `mtx` and `dist` using `cv2.calibrateCamera` method which took the earlier calculated two variables `objpoints` and `imgpoints` as the inputs.
- Lastly, i have undistorted one of the testing images using `cv2.undistort(...)` with the image, `mtx` and `dist` as the inputs, and printed out the results together with the original image side by side (the example is showed below).

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I have performed the distortion-correction using `cv2.undistort` on one of the testing images, an example is shown below:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I have defined a function `binary_image` which returns a binary image combining the following two:
- binary thresholding the result of applying the `Sobel operator in the x direction` on the original image.
- binary thresholding the `S channel (HLS)` on the original image 

An example of the binary image is shown below.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

- First of all, I need to calculate the transform matrix M (and its inverse Minv). To make it possible, I have chosen one of the test images which shows straight lane lines in it and applied `cv2.undistort(...)` in order to get the source image `img_src`. 
- Secondly, I have picked four points from both the source image and the destination image, respectively. 
- Next, `cv2.getPerspectiveTransform(...)` was used to calculate the transform matrices M and Minv, respectively.
- Lastly, an example was provided below for transforming the source image to a "birds-eye view" image, in which straight lane lines were drawn (in <font color='red'>red</font> solid lines) on both images. 

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

- Firstly, I have chosen one of the test images with curved lane lines in it, and then applied pre-processing including distortion-correction, binary image and perspective transformation.
- Then, I have defined a function `find_lane_pixels(...)` which uses sliding windows to find all lane pixels for both left and right lanes respectively. Three hyperparameters were defined as well namely `nwindows`, `margin` and `minpix` which indicate the number and width of the sliding windows as well as the minimum pixels found to re-center window.
- Next, I have defined another function `fit_polynomial()` which obtains the polynomial coefficients for both left and right lane lines. Points of the fitted lane lines are also part of the returns for the next step use. 
- Finally, I have applied the `fit_polynomial()` function to the image pre-processed earlier, and showed the resulting image with sliding windows, lane pixels identified and the fitted polynomial lines accordingly.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I have defined a function `measure_curvature_position(...)` which takes the polynominal coefficients and calcuate the curvatures of the two lane lines as well as the position of the car:
   - conversion has been done to translate pixels to real world meters for both $x$ and $y$ axes. 
   - position of the car was calculated by the difference between the middle point of the two lane line ends and the middle point of the image

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

- Firstly, I have defined a function `warped_back_points()` which calculates back the coordinates of the points in the "bird-eye view" to the original image view.
- Then, I have used this function to convert points fitting for both left and right lane lines back to the original view, combined them together and drawn a polygon on to a black image.
- Lastly, I have added the polygon image with the ordinal image using `cv2.addWeighted(...)` function. the output is shown below.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I have created the framework for outputting the video stream, taken the pipeline function defined earlier. The resulting video is linked [here](https://youtu.be/9eNAkmtv1G8) for reference.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

- the binary image generated still has some noisy background points there, one potential solution might be using different gradient/color channels or threshold values to fine-turn it.
- Another point is regarding the polynomial fitting for lane lines. I observed from the video that some of the curves might be overfitted with surrounding points. the noisy background points might contribute to this; However, I think the choosing of sizes the sliding window and min. points would contribute more. Further fine-turning of these hyperparameters might be one solution.
