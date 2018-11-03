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


[image1]: ./camera_cal/calibration1.jpg "distorted"
[image2]: ./camera_cal/undistorted_images/0.jpg "Udistorted"

[image3]: ./output_images/test_undist/3.jpg "Road undistorted"
[image4]: ./output_images/test_unwarped/3.jpg "Road unwarped"
[image5]: ./output_images/test_all_thresh/3.jpg "Thresholded"
[image10]: ./output_images/test_all_unwarped/3.jpg "Thresholded unwarped"

[image6]: ./output_images/test_undist/1.jpg "Before warping"
[image7]: ./output_images/test_unwarped/1.jpg "After warping"

[image8]: ./output_images/prespective_lines_detected/3.jpg "Sliding window"

[image9]: ./output_images/lane_detected/3.jpg "After warping"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Camera Calibration

#### 1. Briefly state  of computing the camera matrix and distortion coefficients. Providing an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./camera calibration.ipynb"  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained 

Note: i saved the camera calibration and distortion coefficients in a pickle file dictionart to use them in the main file project

this result: 

# Distorted image

![alt text][image1]

# Undistorted image

![alt text][image2]


### Pipeline (single images)

#### 1.An example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

# Undistorted road image

![alt text][image3]

# prespective transformed image

![alt text][image4]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at one function in a cell just after markdown cell contain "here combination of color and gradient thresholds in one function" in `Advanced Lane Finding.ipynb`).  Here's an example of my output for this step.  (note: this is from one of the test images)

i used information extracted from (gradx_thresh), (magnitude and direction) and (color_thresh from s channel and gray image)
Note: i used gray threshold to pass only values above 20 because all values below 20 represent dark shadows

![alt text][image5]
![alt text][image10]



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

(thresholding steps in `warping.ipynb`).

I chose the hardcode the 4 source and destination points in the following manner:

```python
y2 = 475
y1 = 700
src = np.float32([(240, y1), (559, y2), (725, y2), (1072, y1)]) 

marginx1 = src[0][0] + 125
marginx2 = src[3][0] - 125
pady = 400
dst = np.float32([(marginx1, shape[0]), (marginx1, pady), (marginx2, pady), (marginx2, shape[0])]) 
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 240, 700      | 365, 720      | 
| 559, 475      | 365, 400      |
| 725, 475      | 947, 400      |
| 1072, 700     | 947, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]
![alt text][image7]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial

first i used sliding window algorithm to extract only the points related to the lane lines
then i passed those points to function np.poly fit to get the polynomial coefficients of the second order like this

```python
    left_fit = np.polyfit(lefty, leftx, 2)
    right_fit = np.polyfit(righty, rightx, 2)
```


(those steps are in a number of cells just after markdown cell contains "here a function that use sliding window to extract only the points related to the lane line followed by a cell to test it" in `Advanced Lane Finding.ipynb`). 


![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

i calculated radius of curvature based on polynomial coefficients and the y value of the bottom and also calculated offset from center
i did this step in a funcrion called "get_lane_points_and_curvature" 

the code is just after markdown cell contains 
"here a function that use the points extracted to fit a polynomial and get its points and to calculate the curvature and offset from center"

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the cell just after markdown cell contains "here i plotted back down the result onto the road such that the lane area is identified clearly" in `Advanced Lane Finding.ipynb`). 

![alt text][image9]


#### 7. i used a class to save old values of the data to make the onformation in the video move smoothly 



---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---


### Discussion
1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

the main problem i faced is to extract the prespective transform points.
i really think that any transform will change the dimention of the image and that also would affect the values ym_per_pix and xm_per_pex as the lina length in pixels is dependent on how i made the transform
i think my pipeline could fail if it faced a big sequence of untrusted frames 
i think a more improvement in the thresholding, prespective parameters and classes would help 
