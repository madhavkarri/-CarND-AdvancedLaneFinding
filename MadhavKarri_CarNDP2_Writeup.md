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

Python Code/Implementation: [Link](./MadhavKarri-Project2-Files/Pipeline_Test_Images.ipynb)

* Load all neceaary python imports (cell: 1)
* Implement Camera Calibration (cell: 2)
  - Camera calibration was performed using provided chess board images
  - Recommended number of images for a good calibration is about 20
  - However, few of the provided images, images 1,4 and 5, did not fit/match 9X6 pattern
  - Sequence of steps followed for camera calibration:
    - Convert image to grayscale
    - Find corners of chessboard using cv2 function "findchessboardcorners"
    - Create object points in 3d plane and image points. Image points are corners of the chessboard determined in the previous step
    - Calibrate camera using object points and image points using cv2 function "calibrateCamera"
    - "calibrateCamera" function outputs camera calibration matrix and distortion coefficients
    - The original image can be undistorted using camera calibration matrix and distortion coefficients
    - Calibration error can be computed using cv2 function "projectionPoints", and taking L2-Norm between the image poinst and output from cv2 function "projectionPoints"
    - The error initially is quite low on the order of magnitude 10^-2, but increases proprotional to the number of images and consequently will be asymptotic. 
    - The calibration error for the given test images inside the "camera_cal" folder is about approximately 2
    - The resulting camera matrix and distortion coefficients are as follows
    ![CameraCal_CMDC](./Writeup_IV/CameraCal_CMDC.jpg)
* Apply camera matrix and distortion coefficients to undistort each frame (function: frame_ud)
* Apply color selection binary by thresholding S-channel of HLS color space (function: hls_select)
  - HLS Color Space with S-Channel was determined to be the appropriate binary for detecting yellow lane lines under varying brightness or shades
- Apply gradient, gradient-magnitude, and gradient-direction thresholds using sobel operator
  - Gradient Threshold (function: grad_thresh)
  - Gradient Magnitude Threshold (function: mag_thresh)
  - Gradient Direction Threshold (function: dir_thresh)
  - Gradient-x and y will result in highlighting line-edges features closer to vertical and horizontal directions respectively
* Apply masking and perspective transform (function: frame_mptw)
  - Masking was applied by picking 4-corner co-ordinates of a trapezoid close enough to the target area on the lane
* Find lane starting co-ordinates on X-axis by taking histogram of detected lane line pixels on bottom-half of the warped binary output (function: wi_hp)
  - A histogram of the lane line pixels on the bottom half of the frame/image in the target area will result in two peaks. These two peaks, corresponding to left and right lane lines, can be used as a good starting guess for the sliding window algorithm.
* Post establishing the starting co-ordinates, of the detected lane line pixels, on x-axis, implement the sliding window algorithm to detect lane line pixels through all of the image/frame (function: find_lane_pixels)
* Post detetction of left and right line pixels through the complete image, a polynomial fit is established for each of the left and right lines (function: fit_polynomial)
* Implement band/margin search around line position from previous frame, equivalent to using a customized region of interest for each frame of video. In the unlikely case, lost track of the lines, revert back to sliding windows search (function: search_around_poly)
* Measure radius of curvature for the lane and offset of center of vehicle from center of lane (function: measure_curvature_real)
* Final Step: Map/Draw detected lane on to the original undistorted frame/image (function: map_lane_udi)

The above set of steps were implemented on the following set of test images
* Test1.jpg
* Test2.jpg
* Test3.jpg
* Test4.jpg
* Test5.jpg
* Test6.jpg
* Straight_lines1.jpg
* Straight_lines2.jpg

Results from Pipeline for test images on Test3.jpg are shown below:
* Applying camera matrix and distortion coefficients to undistort each frame
![Test3_Undistorted](./Writeup_IV/Test3_Undistorted.jpg)
* Apply color selection binary by thresholding S-channel of HLS color space
![Test3_HLSBinary](./Writeup_IV/Test3_HLSBinary.jpg)
* Apply gradient, gradient-magnitude, and gradient-direction thresholds using sobel operator
![Test3_GMDBinary](./Writeup_IV/Test3_GMDBinary.jpg)
* Apply masking and perspective transform
![Test3_MPTBinary](./Writeup_IV/Test3_MPTBinary.jpg)
* Find lane starting co-ordinates on X-axis by taking histogram of detected lane line pixels on bottom-half of the warped binary output
![Test3_HPeaks](./Writeup_IV/Test3_HPeaks.jpg)
* Implement sliding window algorithm to detect lane line pixels through all of the image/frame 
* Perform a polynomial fit on the detetcted lane line pixels for each of the left and right lines
![Test3_SWPF](./Writeup_IV/Test3_SWPF.jpg)
* Implement band/margin search around previously detected left and right lines from previous frame
![Test3_SAPC](./Writeup_IV/Test3_SAPC.jpg)
* Measure radius of curvature for the lane and offset of center of vehicle from center of lane
![Test3_CurvRad](./Writeup_IV/Test3_CurvRad.jpg)
* Final Step: Map/Draw detected lane on to the original undistorted frame/image (function: map_lane_udi)
![Test3_LDetect](./Writeup_IV/Test3_LDetect.jpg)

**Finding Lane Lines in a Video (Project_Video)**

- The pipeline/steps implemented for finding lane lines in a video are similar to finding lane lines on test images
  - Implemented a python code to extract raw frames/static images from a video file
  - Implemented a pipeline consisting primarily of 1 main function and all the functions previously listed in the descripition of finding lane lines on test images.
  - Main Function/Wrapper: A while loop that calls all the necessary functions repeatedly on each of the image frames extracted from the original video.
  - Implemented a python code to stich final output frames from the preceeding steps and convert it into a video
  
Additional Comments
- A line class has been implemented which keeps tracks of all the line charachteristics (fit coefficients, radius of curvature, vehicle center offset, etc.) between frames
- An attempt has been made to avergae the polynomial fit coefficients  between n frames. The averaging has been performed such that for the nth frame, frames from n-10 to n+10 were used to average the polynomial coefficients.
- Smoothing: the objective of the averaging technique was to minimize the jumping of lane lines from frame to frame
- Surprisingly, the video output without averaging performed better than the one with averaging. Likely cause might be the number of frames being considered for averaging might be too high
- The below final output video is one without averaging

**Project Video Output**

 - Project Video
   - Python Code/Implementation: [Link](./Writeup_IV/Pipeline_Project_Video.ipynb)
   - Video Output File (YouTube): [Link](https://youtu.be/wEArUuQCSfM)

---

### Optional Challange

- The pipeline/steps implemented for finding lane lines for the challenge video are similar to finding lane lines on the project video
  - Implemented a python code to extract raw frames/static images from a video file
  - Implemented a pipeline consisting primarily of 1 main function and all the functions previously listed in the descripition of finding lane lines on test images.
  - Main Function/Wrapper: A while loop that calls all the necessary functions repeatedly on each of the image frames extracted from the original video.
  - Implemented a python code to stich final output frames from the preceeding steps and convert it into a video
  
Additional Comments
- First major challenge in this video was to detect/identify and remove the misealding/spurious lane markings as shown below
![ChallengeVideo_MSLM](./Writeup_IV/ChallengeVideo_MSLM.jpg)
- The misealding/spurious lane marking are detected using HSV colorspace. By adjusting the thresholds, the HSV binary was able to detect all the lanes and its features except for the yelllow and white lane markings. The result is shown below
![UD_HSV17](./Writeup_IV/UD_HSV17.jpg)
- The solution was to subtract the HSV binary from the undistorted image to remove the misealding/spurious lane markings
- The subtraction was performed post applying   - Gradient, Gradient Magnitude, and Gradient Direction Thresholds


**Challenge Video Output**

 - Challenge Video
   - Python Code/Implementation: [Link](./Writeup_IV/Pipeline_Challenge_Video.ipynb)
   - Video Output File (YouTube): [Link](https://youtu.be/jzV-YqbOaFw)
   - Video Output File: [Link](./Writeup_IV/ChallengeVideo_fo.mp4)
  
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


