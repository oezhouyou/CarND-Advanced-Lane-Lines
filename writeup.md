
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

[image1]: ./output_images/undistort.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary_combo_example.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/example_output.jpg "Output"
[video1]: ./output_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Since OpenCV library has pretty mature functions handling camera calibration and image undistorting. I will use those exsiting tools to do this task. First, cv2.findChessboardCorners( ) is used to get objpoints and imgpoints. Then cv2.calibrateCamera( ) uses those points to compute the camera calibration and distortion coefficients. Finally, cv2.undistort( ) outputs the undistorted images by taking advantage of those coefficients.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of H channel and L channel Sober gradient thresholds to generate a binary image.  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Two opencv functions cv2.getPerspectiveTransform( ) and cv2.warpPerspective( ) are used to perform perspective transform. Function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
    src = [[100, img_size[1]],
           [img_size[0] * 0.45, img_size[1] * 0.63],
           [img_size[0] * 0.55, img_size[1] * 0.63],
           [img_size[0] - 100, img_size[1]]]

    dst = [[src[0][0] + offset_x, img_size[1] + offset_y],
           [src[0][0] + offset_x, 0 + offset_y],
           [src[-1][0] - offset_x, 0 - offset_y],
           [src[-1][0] - offset_x, img_size[1] - offset_y]]
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

A sliding window is used to accumulate the lane pixels from bottom of the image to the top, checking out below 2nd order polynomial:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

For the radius of curvature, np.polyfit( ) is used first to generate left and right lane curvature. Afterwards, np.average( ) generates the averaged radius.

For the position with respect to center, the x location of each lane bottom pixel is known in previous step. The average of those two x location is the center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In order to generate thresholded binary image, multiple approach like color transforms, gradients and combination solutions are tried, which is quite time consuming to fine tune the perimeters. 

When the vehicle is doing lane change, the pipeline would probably fail. In that situation, the algorithm should be able to detect any number of lanes and match each lanes to the HD map, to better aware of surrounding.  
