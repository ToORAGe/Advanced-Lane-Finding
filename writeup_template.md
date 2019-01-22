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

[image1]: ./output_images/Original_chessboard.jpg "Original Chessboard"
[image2]: ./output_images/Undistorted_chessboard.jpg "Undistorted Chessboard"
[image3]: ./output_images/original_test.jpg "Original image (distorted)"
[image4]: ./output_images/undistorted_test.jpg "Undistorted image"
[image5]: ./output_images/undistorted_nograds.jpg "Undistorted image"
[image6]: ./output_images/binary_gradsandcolors.jpg "Binary threshold image"
[image7]: ./output_images/undistorted_test.jpg "Undistorted image"
[image8]: ./output_images/warped_nograds.jpg "Binary threshold image"
[image9]: ./output_images/final_output.jpg "Final result"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "sample_code.ipynb" 
This notebook was only used to find the distortion matrices(mat and dist) and perspective warping matrices (M, Minv). 
For the rest of the project the Project2_fullcode.ipyn notebook has been used. 
I start by reading in the images using the glob API. These images are then stored in a list. Corners in these calibration images are then found out using the cv2.findChessboardCorners function. 
These corners are stored along with the object points (x,y,z) in two separate arrays img_pts and obj_pts. 
These arrays of points are used to compute the distortion matrices using the cv2.calibrateCamera funciton. 
The images are then undistorted using the cv2.undistort function.

The input and output images are as follows.
![alt text][image1]
![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The undistorted images obtained from the camera calibration are as follows 


![alt text][image3]
![alt text][image4]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used the S channel thresholding and gradient magnitude and direction thresholding to obtain a binary image. 
The threshold binary image for one of the test images is shown below. 

![alt text][image5]
![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

To perform a perspective transform I had to first decide on what points I would be using for the transformation. For this task I chose the straigh line test image provided and located the boundary points of the lane lines. And these source points were 
src =  np.float32([[80,imshape[0]],[490,490],[790,490],[1200,imshape[0]]])

Now for the destination points I wanted the lane lines to be straight on the warped image hence chose the same x coordinates for the two points on the upper end of the lane (top of the image) as that of the points on the bottom. 

dst = np.float32([[80,imshape[0]],[80,0],[1200,0],[1200,imshape[0]]])

The following are the images before and after warping. 


![alt text][image7]
![alt text][image8]


Afetr obtaining the M and Minv matrices I have hardcoded these matrix values to the process_image function for processing the video to save time instead of calibrating the camera everytime the video pipeline is called for the video processing. (Not recommended) 
#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The lane_recognition function takes care of identfying the pixels of the image which form the lane lines.

I  use the warped and binary thresholded image as the starting point. 

Using the sum feature I find the positions of the lane lines by searching for the two maxima on the left and right respectively. 

With these positions I start a sliding window search and track all the pixels. If the pixels found in a current window are greater than minipix the window position is moved to the mean of those pixels or is kept the same. 

A list is prepared containing all the indices of the image where the lane lies are located. 

After finding the indices I fit polynomials through the points using the polyfit function. 

I unfortunately could not view the images from this function. Searched online for any possible reasons and found that the version of matplotlib could be to blame. The output however could be utilized without any errors in the remaining part of the pipeline.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature was found based on the formula described in the lecture notes and the code for the same can be found in the 'curvature_finding' funciton of the 'Project2_fullcode.ipyn'

The polynomial coefficients and the coordinates were converted to metres using the xm_per_pix and yym_per_pix parameters and 
the curvature_finding function is called to find the radius of curvature of the lane lines at y = bottom of the image in metres. 

The position of the vehicle with respect to the center of imaage was found by subtracting the center of the image from the center of the bottom part of the lane lines. Decide on the side (right or left) based on +ve or -ve value of the difference. 



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The pipeline is coded in the Project2_fullcode.ipyn. This pipelines takes in the raw image and converts it to the resultant image with lane lines drawn onto it. 

Example image is shown below 

![alt text][image9]


### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

The final output project_video.mp4 is located in the output_images folder. 

A class was created to track the polynomial fits of the lines and also their coordinates. 
In case the pipeline fails (lane lines are not detected properly) the class objects are called to compute the average polynomials for that frame. 

The criter of pipeline failing is set as if the minimum distance between the left and right lane lines is less than 80% of the intended width of the lanes (1000 pixels in my case) or if the curvature of the lanes is opposite in direction. 


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Issues with the pipeline
a) Does not perform well when dark patches are present on the road (shadows of trees) (binary thresholding fails to detect lanes properly) 
b) The src and dst points need to be defined for each video separately, else the pipeline seems to underestimate or overestimate the boundaries.
c) Takes long time to compute if the distortion and warping matrices are computed afresh. 
d) In order to make the pipeline more robust I would use additional colorspace thresholding to separate out shadows from lane lines. 
e) A better averaging would have results in smoother results for the lane lines when the detection by pipeline failed. 


Issue particular to my pipeine 
a) Does not output images for the binary image with lane lines. Version of matplotlib could be root cause. 
Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
