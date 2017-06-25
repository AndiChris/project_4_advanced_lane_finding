## Writeup Template

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

[image1]: ./output_images/calib_undistorted.jpg "Undistorted"
[image2]: ./output_images/example_undist.jpg "Road Undistorted"
[image3]: ./output_images/example_binary.jpg "Binary Example"
[image4]: ./output_images/bird_eye_orig.jpg "Warp Example"
[image5]: ./output_images/bird_eye_warped.jpg "Warp Example"
[image6]: ./output_images/lane_px_with_fit.jpg "Fit Visual"
[image7]: ./output_images/test_with_lane_1.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3. code cell of the IPython notebook located in "./code.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images. Resulting in this:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color, gradient and direction thresholds to generate a binary image (thresholding steps at lines 36 through 39 in code cell 5).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in code cell 7th of the IPython notebook.  The `perspective_transform()` function takes as inputs an image (`img`), as well as a reverse flag (`reverse`) which will revert the whole transformation. I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32([[570,460], [720,460], [1150,720], [170,720]])
offset = 100
dst = np.float32([[offset,0],
                  [img.shape[1]-offset, 0],
                  [img.shape[1]-offset,img.shape[0]],
                  [offset,img.shape[0]]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 570, 460      | 100, 0        |
| 720, 460      | 1180, 0       |
| 1150, 720     | 1180, 720     |
| 170, 720      | 100, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image5]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the 9th cell I defined a line class to bundle all information regarding the line detection. The `search_line_in_warped_img()` function (line 33 - 50) serves as an entry point. The processing pipeline has to call this function for each lane (left and right). The function takes as an argument a binary image `binary_warped_img` and a flag to specify whether it is a left or right lane.
This function decides based on the `self.detected` variable if we shall detect lines from ground up via a histogram/sliding-window search or if it can simply infer lane pixels from a previous iteration.

The `__find_fresh_line()` function (line 52 - 128, cell 9) first makes a histogram of the binary-image and based on the peak (line 57/59) it searches with a sliding-window algorithm for corresponding pixels (line 79-92). At line 104 a 2nd order polynomial is fitted against all found pixels coordinates. The following image demonstrates this by coloring all found pixels blue and red and the fitted curve in yellow.

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In code cell 9 at line 204 in the function `__calc_radius()` I calculated the curvature of the lane. For the position of the vehicle with respect to center I used a combination of first storing the relative value of each line at line 172 (still code cell 9) and then in code cell 11 line 38 I used both values (from left and right line) to calculate the final position.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 11 in the function `draw_lines_to_img()`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced some major issues with only using color threshold in the HLS color-space. Somehow to much noise from shadows were picked up confusing the detection. After adding HSV additionally the detection become more robust.

To increase robustness of the pipeline I could add more sanity checking for the values. Especially when a line object is assigned to be specific left or right it could make further checking in each iteration. Once I faced the problem that the detection got messed up an the right line class object "jumped" over to the left leading to two objects detecting the same line.

Of course another problem is probably if another car crosses lines in front of our car hiding the lane markings partially. Then I guess the program will have major difficulties.
