## Lane Line Detection for Autonomous Cars
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](https://www.udacity.com/course/self-driving-car-engineer-nanodegree--nd013)

In this project, the goal is to write a software pipeline that detects the lane on the road in videos taken from the front camera of an autonomous car with the OpenCV library in Python, using techniques such as Camera Calibration, Distortion Correction, Perspective Transform, Color and Gradient Thresholding, etc.

The steps of the pipeline are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The images for camera calibration are stored in the folder called `camera_cal`.  
The images in `test_images` are for testing the pipeline on single frames. 
Examples of the output from each stage of the pipeline are saved in the folder called `ouput_images`. 
