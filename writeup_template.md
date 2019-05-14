## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is in the function getCameraAndDistCoeff(). This function takes the input images directory as the input parameter. The camera images are read one by one from the input directory passed and the corners are detected using the function cv2.findChessBoardCorners(). The corners detected using the above function are then passed to the function cv2.drawChessBoardCorners() to draw the lines across the chess board corners.

Initially the object points are prepared. Object points are nothing but the x, y and z coordinates of the chess board. We assume that the z coordinate is zero and only consider the x and y plane of the chess board. The image points represent the x and y plane of the chess board. Once the corners of the chess board are detected, they are appended to the image points list created. The objpoints and the imagepoints are then passed to the function cv2.calibrateCamera() to calibrate the images. The output of this function are the distortion co-efficients and camera matrix which are the properties of the lens and camera respectively.

The distortion co-efficients and the camera matrix are then passed to the function undistorCameraImages(). This
function undistorts the camera images using the function cv2.undistort(). The undistorted camera images are stored in
the folder camera_calibrated_output_images.

Similarly, the test input images which are in the folder test_images are also undisorted in the similar fashion. The
undistorted test images are stored in the folder output_images directory.


#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

After the test images and the camera images are undistorted, they are warped. The warping of the camera images is in the
function warpUndistortedCameraImages() and the function warpUndistortedTestImages() respectively. The input parameters
passed to these functions are the undisotrted images, distortion coefficient and camera matrix obtained from the
previous step.

In this step, the source and the destination points are chosen. Both the source and destination points are chosen
randomly by just looking at the image. This is the step that took a lot of time. It was very difficult to get to the
correct source and destination points.

The source and destination points chosen are :
src = np.float32([[570, 450],
		 [750, 450],
		 [1100, 660],
		 [270, 660]
		])

dst = np.float32([[210, 0],
		 [1070, 0],
		 [1070, 720],
		 [210, 720]
		])

After choosing the src and the dst points, the function cv2.getPerspectiveTransform() is called to warp the images.

The warped output images for camera images are put in the folder camera_calibrated_undistorted_warped_output_images.
The warped output images for test images are put in the folder output_warped_images.


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

After the images are warped, they are converted into the thresholded binary images using various techniques. The
function convertWarpedImageToBinaryImage() has the functionality of converting the warped image to the binary image.
This step takes the input image as the warped image obtained from the previous step.

For every image, the different thresholding techniques applied are absolute thresholding (absSobelThresh() in the code),
magnitude of gradient (magnitudeOfGradient() in the code), direction of gradient (directionOfGradient() in the code)
and color thresholding (colorThresholding() in the code).

In the absolute thresholding technique, sobel operator is applied in the x direction. This way, only the vertical pixels
are chosen. The thresholding parameters applied are thresh_min = 20 and thresh_max = 100. Only the pixels between this
range are chosen.

In the magnitude of gradient technique, the sobel operator is applied in both the x and y directions and the magnitude
across both the directions are taken. The thresholding parameters applied in this technique are, thresh_min = 20 and
thresh_max = 120.

In the direction of gradient technique, the sobel operator is again applied in the both the directions and to these, the
function np.arctan2() is applied to get the direction of the gradient. The thresholding parameters applied in this case
again are, thresh_min = 0.7 and thresh_max = 1.3.

In the color thresholding technique, the image is first converted to the HLS image space, and from that, the s channel
is chosen. The s channel is chosen because this is the stable channel and the brightness remains the same irrespective
ofthe change in the brightness. The thresholding parameters used in the this case are thresh_min = 90, thresh_max = 255.

All the above are combined and a combined binary image is created. The combinedBinaryImage() consists of this
functionality.

The output of warped binary images can be found in the folder output_color_gradient_binary_images.


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The images obtained from the previous step are then passed as a parameter to the function
findTheLanePixelsAndMeasureRadOfCurv(). As mentioned, this function takes warpedBinaryImages as input parameter.

4.1 	This function inturn calls the function fitPolynomial(), which in turn calls the function findLanePixels which inturn
	calls the function getHistogram. All this is done to get the left and and right polynomials, i,e, get the polynomials
	that fit the left and the right lane.

	In the findLanePixels() function, sliding window method is used to get the polynomials of both the left and the right
	lane. The left and right lane lines are divided into the 9 windows and the good pixels are calculated for the
	left and the right lanes.

	If there is only one frame, which is mainly in the case of images, the sliding window method is used to
	calculate the left and the right polynomials.

4.2 	If there are more than one frame, which is mainly in the case of video, the function searchAroundPoly() is called to
	calculate the left and the right polynomials. The polynomials obtained in the step 4.1 are passed to this
	function to get the new set of polynomials for the next frame.


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function measureRadiusOfCurvature() contains the code to calculate the radius of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The function projectLinesOnOriginalImage() consists of the code where the lane lines are drawn on the images. The final
output images are stored in the folder final_output_images.

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The pipeline for video is in the process_image(). To undisort the frames in the video, the function cv2.undistort() is
used. The warp the undistorted video, the function warpVideoFrame() is used. To convert the warped image to the binary
image, the function convertWarpedVideoImageToBinaryImage() is used.

To find the polynomials for the very first frame in the video, the function slidingWindowStepForVideo() is used and to
find the polynomials for the susequent frames in the video, the function searchAroundPoly() is used. 

The variables in the class Line are updated for both the left and the right line.

Finally, drawing the lane lines on the video and measuring the radius of curvature are done in the functions
projectLinesOnVideoImage() and measureRadiusOfCurvature() respectively.

The output of the project_video is stored in the folder final_output_videos.

The output of the challenge_video is stored in the same folder where the challenge_video is present. The name of the
output video is project_video_output.

The output of the harder_challenge_video is stored in the same folder where the harder_challenge_video is present. The
name of the output video is harder_challenge_video_output.

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced the following problems:

1. Finding the src and dst points took way more time than I expected. I somehow got to the src and dst points mentioned
   above in the document. 
2. I could have reused the same functions for both the image and the video pipeline. I could not make it because I did
   not design the project properly.
