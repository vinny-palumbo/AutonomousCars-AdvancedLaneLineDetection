## Lane Line Detection for Autonomous Cars

---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify the binary image ("birds-eye view").
* Detect lane pixels and fit a polynomial on the lane lines.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibration9.jpg "Original Calibration Image"
[image2]: ./output_images/calibration9-undistorted.jpg "Undistorted Calibration Image"
[image3]: ./output_images/straight_lines1.jpg "Original Test Image"
[image4]: ./output_images/straight_lines1-undistorted.jpg "Undistorted Image"
[image5]: ./output_images/straight_lines1-gradientThresh.jpg "Gradient Thresholded Image"
[image6]: ./output_images/straight_lines1-colorThresh.jpg "Color Thresholded Image"
[image7]: ./output_images/straight_lines1-combinedThresh.jpg "Gradient & Color Thresholded Image"
[image8]: ./output_images/straight_lines1-warped.jpg "Warped Image"
[image9]: ./output_images/straight_lines1-slidingWindow.jpg "Sliding Windows"
[image10]: ./output_images/straight_lines1-laneProjection.jpg "Lane Projection"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 6th code cell of the IPython notebook pipeline.ipynb

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world: (0,0,0), (1,0,0), (2,0,0), ..., (8,5,0). Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a calibration image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function.

Here are examples of the original (left) and undistorted (right) versions of one of the calibration images: 

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here are examples of the original (left) and undistorted (right) versions of one of the test images:

![alt text][image3]
![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradient and color thresholds to generate a final binary image. 

For the Gradient Thresholds, I used a combination of gradient magnitude (11th code cell in notebook) and gradient direction (12th code cell in notebook) thresholds. Here are examples of the original (left) and gradient thresholded (right) versions of one of the test images:

![alt text][image4]
![alt text][image5]

For the Color Thresholds, I used a combination of saturation-channel (14th code cell in notebook) and red-channel (15th code cell in notebook) thresholds. Here are examples of the original (left) and color thresholded (right) versions of one of the test images:

![alt text][image4]
![alt text][image6]


Finally, I combined the Gradient and Color thresholds to generate the final binary image. Here are examples of the original (left) and gradient & color thresholded (right) versions of one of the test images:

![alt text][image4]
![alt text][image7]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform appears in the 19th, 20th and 21th code cells of the IPython notebook. I used the `cv2.warpPerspective()` function, which takes as inputs an image, as well as a perspective transform matrix (M). In this case, the input image is the binary image output in the previous step. The M matrix in calculated using the `cv2.getPerspectiveTransform()`, which takes as inputs source (`src`) and destination (`dst`) points. I chose to hardcode the source (driver perspective) and destination (bird eye persepective) coordinates of the calibration box in the following manner:

```python
# driver perspective
src = np.float32(
    [[195,720], # bottom left
     [1117,720],  # bottom right
     [695,454], # top right driver
     [587,454]]) # top left driver
# bird eye perspective
dst = np.float32(
    [[195,720], # bottom left
     [1117,720],  # bottom right
     [1117,0], # top right
     [195,0]]) # top left
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test binary image with straight lines and its warped counterpart, to verify that the lines appear parallel in the warped image.

![alt text][image7]
![alt text][image8]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step of my pipeline appears in the 23th and 24th code cells of the IPython notebook.

In the `slidingWindowSearch()` function, I first plot a histogram summing the pixels' value of all the columns in the lower half of the warped binary image. I then find the peak's x-axis position of the left and right halves of the histogram. These are the starting point for the left and right lines. I use sliding windows placed around the line centers to detect the lines' pixels in those windows up to the top of the frame. Finally, I fit a 2nd order polynomial to each of the left and right lane lines' detected pixels.

I tested this step by plotting the lines polynomial fits, sliding windows and detected pixels on a test binary image:

![alt text][image9]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to the center.

The code for this step of my pipeline appears in the 28-31th code cells of the IPython notebook.

To measure the lane's Radius of Curvature, I applied the formula to calculte it at the bottom of the image (where the car is positionned on the lane) for each of the second-order polynomial curves. I made sure I converted the coordinates from pixels space to real-world space (in meters), knowing that the lane is about 30 meters long and 3.7 meters wide. I finally simply averaged the radius of curvature of the 2 lane lines to get the final radius of curvature of the lane.

To measure the position of the vehicule with respect to the lane center, I first calculated the x-position of the center of the images (which corresponds to the center of the vehicule). I then calculated the x-position of the center of the lane at the bottom of the image (where the car is positionned on the lane) by calculating the midpoint between the two polynomial curves. The difference between those 2 x-position values gives the relative position of the vehicule to the lane center. Again, I converted the coordinates from pixel space to real-world space. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step of my pipeline appears in the 32-33th code cells of the IPython notebook. 

In the `projectLaneAndWriteInfo()` function, I basically drew green pixels between the 2 polynomial curves in the bird eye perspective and warped that back to the original driver perspective using the inverse perspective matrix (Minv).

Here is an example of the lane being projected in green on a test image:

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

![alt text][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline includes color thresholds on the saturation-channel (good for detecting yellow lines) and on the red-channel (good for detecting white lines). To make the pipeline more robust and detect the lane lines at night, it would be good to use a technique that neutralizes low-brightness images. Also, finding a way to eliminate the noise from the windshield reflection could be a good improvement.

Sometimes, there are no lane lines on the road. The current pipeline couldn't project the lane and calculate its curvature in such cases. Also, it would have a hard time if there are multiple lines on the lane.

To make it faster, the pipeline should not just search blindly for the lane lines in each frame of video, but rather, once its has a high-confidence detection, use that to inform the search for the position of the lines in subsequent frames of video. For example, if a polynomial fit was found to be robust in the previous frame, then rather than search the entire next frame for the lines, just a window around the previous detection could be searched. This would improve speed and provide a more robust method for rejecting outliers.

For an additional improvement, we could implement outlier rejection and use a low-pass filter to smooth the lane detection over frames, meaning add each new detection to a weighted mean of the position of the lines to avoid jitter.

