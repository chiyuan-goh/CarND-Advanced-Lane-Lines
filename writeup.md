## Writeup Template



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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[output]: ./output_images/output_img.png "Output"
[thres]:  ./output_images/thres.png
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! In addition, I recorded the results of the intermediate steps in `examples/draft.ipynb`, so please take a look at that as well.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `./examples/draft.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained the result shown in the 4th code cell in `examples/draft.pynb`.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color, magnitude, gradient and directional thresholds to generate a binary image on the H,L,S channels in  (thresholding steps at cell 16 in `examples/draft.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][thres]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `get_perspective_transform()`, which appears in the 2nd code cell of the IPython notebook). I chose to hardcode the source and destination points in the following form:


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 738, 492      | 980, 300        | 
| 539, 492      | 300, 300      |
| 307, 660     | 300, 720      |
| 1005, 660      | 980, 720        |

I verified that my perspective transform was working in the 5th and 6th code block in `examples/draft.ipynb`

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The identification of the lane-line pixels and fitting of the 2nd order polynomial is performed in the 10th code cell in `examples/draft.ipynb`.

Essentially, the following steps are performed:

1. After acquiring the perspective transformed binary image, I use a histogram to count the number of 1s across the different columns of the bottom half image.
2. I then note the x positions of the left and right modes where the left and right modes are located on the first/last `img.shape[1]/2` columns respectively. They are the centers of the initial region of interest (ROI). Each ROI have a predefined height and width. 
3. For each ROI, I collect all the valid points inside it and add then to a list.
4. A new x-center for each ROI is calculated by averaging the x-positions of the valid points.
5. I then move up the image and repeat step 3-5 until the top of the thresholded image.
6. Finally, I use all the collected points from the ROIs to fit a 2nd degree polynomial.

For better understanding of the ROI window, please see a visualization on the 11th code block of `examples/draft.ipynb`.   

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of the curvature of the lane markings can be found using the formula provided [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php). So, essentially, once I found the coefficients of the 2nd order polynomial, I just need to apply a simple differentiation to obtain the first and second differentials, and finally use the largest Y value, which is the lane marking closest to the vehicle as the point where this radius of curvature is to be calculated.
 
I did this in the 12th code cell in `examples/draft.ipynb`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 17th code cell in `draft.ipynb` in the function `test_lane_detection()`.  Here 	is an example of my result on a test image:

![alt text][output]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://drive.google.com/open?id=0Bxtv1dvjqwk7MzFsZy1QMmFVbTA)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach used in this implementation starts off with a search on the modes in the histogram. While this works on the provided video, clearly this does not work in all scenarios. Imagine if there is a large vehicle on the next lane. Clearly, the line/edge generated by the edge of the vehicle dominates the actual lane in the histogram because the lane is stripped. This would led to the algorihm believing that the lane is wider that it is. There are also occasions when there are other markings on the road (e.g left/right turn) apart for the lane separators which might confuse the algorithm.

Possible solutions to the first issue could include vehicle detection algorithms to remove the false positive.