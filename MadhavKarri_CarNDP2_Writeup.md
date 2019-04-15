# **Self-Driving Car Engineer Nanodegree**
# **Project2: Advanced Lane Finding**

## MK

The goals/steps for this project are:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels (using histogram peaks) and polynomial fit to find the lane boundary.
* Implement margin/band search based on curve fits from previous frame.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
* Make a pipeline to implement the above set of steps on all of the frames from the video.
* Reflect on your work in a written report

---

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

**Finding Lane Lines on Test Images/Frames**

Pipeline for test images (Interactive python file: Pipeline_Test_Images.ipynb)

* Load all neceaary python imports (cell: 1)
* Implement Camera Calibration (cell: 2)
* Apply camera matrix and distortion coefficients to undistort each frame (function: frame_ud)
* Apply color selection binary by thresholding S-channel of HLS (function: hls_select)
- Apply gradient, gradient-magnitude, and gradient-direction using sobel operator
  - Gradient Threshold (function: grad_thresh)
  - Gradient Magnitude Threshold (function: mag_thresh)
  - Gradient Direction Threshold (function: dir_thresh)
* Apply masking perspective transform, and warping (function: frame_mptw)  
* Find lane starting co-ordinates on X-axis by taking histogram of detected lane line pixels on bottom-half of the warped binary output (function: wi_hp)
* Post establishing the starting co-ordinates, of the detected lane line pixels, on x-axis, implement the sliding window algorithm to detect lane line pixels through all of the image/frame (function: find_lane_pixels)
* Post detetction of left and right line pixels through the complete image, a polynomial fit is established for each of the left and right lines (function: fit_polynomial)
* Implement band/margin search around line position from previous frame, equivalent to using a customized region of interest for each frame of video. In the unlikely case, lost track of the lines, revert back to sliding windows search (function: search_around_poly)
* Measure radius of curvature for the lane and offset of center of vehicle from center of lane (function: measure_curvature_real)
* As a last step map/draw the detected lane on to the original undistorted frame/image (function: map_lane_udi)

The above set of steps were repeated on the following set of images
* Test1.jpg
* Test2.jpg
* Test3.jpg
* Test4.jpg
* Test5.jpg
* Test6.jpg
* Straight_lines1.jpg
* Straight_lines2.jpg

Results for solid-white-right:

Python Code/Implementation: [Link](./MadhavKarri-Project1-Files/solidWhiteRight-Copy1.ipynb)

![WI_SolidYellowCurve24](./Writeup_IV/WI_SolidYellowCurve24.png)

**Finding Lane Lines in a Video**

- Following are the pipeline/steps implemented for finding lane lines in a video
  - Implemented a python code to extract raw fraemes/static images from a video file
![WI_SolidWhiteRightV1](./Writeup_IV/WI_SolidWhiteRightV1.png)

  - Implemented a pipeline consisting primarily of 1 main function and 3 call-back function definitions:
    - Function1: "my_ced" that implements Canny Edge Detection 
      - Input: ".jpg" image file
      - Output: edges from the Canny Edge Detection algorithm
    - Function2: "my_mpht" that implements Masking and P-Hough Transform to get line and line-coordinates.
      - Input: ".jpg" image file, masking coordiantes, and edges output from ("my_ced") Canny Edge Detection algorithm
      - Output: co-ordinates for all the lines detected by P-Hough Transform and consequently "m-slope" and "b-intercept" parameters
    - Function3: "my_fld" that implements drawing final lines and adding transperency
      - Input: ".jpg" image file and co-ordinates of all the lines detected by P-Hough Transform
      - Output: Original raw image/frame overlayed with detected/predicted lane lines
      - Extras: In addition this function also adds trasparency and saves the final output as a ".jpg file"
    - Main Function/Wrapper: A while loop that calls Functions 1, 2, 3 repeatedly on each of the image frames extracted from the original video.

  - Implemented a python code to stich final output frames from the preceeding steps and convert it into a video
![WI_SolidWhiteRightV2](./Writeup_IV/WI_SolidWhiteRightV2.png)

**Final Video Output**

 - Solid White Right
   - Python Code/Implementation: [Link](./MadhavKarri-Project1-Files/Porject1Video-SolidWhiteRight/solidWhiteRight-mp4.ipynb)
   - Video Output File: [Link](./Writeup_IV/solidWhiteRight_fo.mp4)

  - Solid Yellow Left
    - Python Code/Implementation: [Link](./MadhavKarri-Project1-Files/Porject1Video-SolidWhiteRight/solidYellowLeft-mp4.ipynb)
    - Video Output File: [Link](./Writeup_IV/solidYellowLeft_fo.mp4) 
      
---

### Reflection

### Potential shortcomings with the current pipeline
- A lot of manual processing is involved with the exisiting pipeline.
  - Manual picking of masking co-ordinates
  - Fine tuning parameters for each of the Gaussian Blur/Canny Edge Detection/Masking/P-Hough Transform steps, specifically on each individual frames
  - Spurious or unwanted slopes, when segregating slopes to detect line segments associated with each of the left and right lanes

### Possible improvements to pipeline
- Research/Implement detection of arc/curvature line segments available in opencv
- Automate the process either through Machine Learning/AI for the following set of tasks
  - To pick optimal co-ordinates for masking
  - To reject spurious or unwanted slopes during slope segregation to identiyf left and right lanes
- Using mapping and localization explore possibility to detect and utilize priori information on lane coordinates and curvature
- Currently in the videos the lane detection lines are bumpy. The bumpiness can be minimized by adding weights during averaging of slopes. 

### Optional Challange
- Code that produced SolidYellowLeft.mp4 was attempted to detect lane lines. Although, it performed well on a good number of frames, it had issues detecting lane lines over the bridge.
- Tried modifying different parameters and found out converting the original raw image/frame using color space conversion code to hsv within cv2.cvtcolor resulted in optimal detection of the yellow and white lanes through all the frames in the video
- Solid Yellow Left
  - Python Code/Implementation: [Link](./MadhavKarri-Project1-Files/Porject1Video-Challenge/Challenge-mp4.ipynb)
  - Video Output File: [Link](./Writeup_IV/Challenge_fo.mp4)

