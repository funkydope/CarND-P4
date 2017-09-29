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

[image1]: ./output_images/calibration1_undist.jpg "Undistorted Chessboard"
[image2]: ./output_images/straight_src.jpg "Straight src"
[image3]: ./output_images/straight_dst.jpg "Straight dst"
[image4]: ./output_images/test1_undist.jpg "Test1 undist"
[image5]: ./output_images/test1_binary.jpg "Test1 binary"
[image6]: ./output_images/test1_curve_fit.jpg "Test1 Curve Fit"
[image7]: ./output_images/test1_detected.jpg "Test1 Detected"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P4-Advanced_Lane-Submit.ipynb"

Images from the camera must be undistorted to ensure that they properly represent real world dimensions. The camera lens creates distortion. We use image processing to correct this.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion Correction.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image4]

#### 2. Create Binary Threshold Image

A combination of HSV and x sobel thresholding worked best to find lines in the video. HSV worked best. I created threshold values for white and yellow lane lines separately. I then used a binary or operator to combine the 2 thresholds to find both white and yellow lane lines. Sobel x was worked better than other sobel derivatives but was more noisy than using color alone. However, I found that I needed it because using color alone wasn't robust enough for the video.

![alt text][image5]

#### 3. Top Down Perspective Transform

The Open CV perspective transform function requires 4 image points source and destination. I used the straight line image from the test images to reference the source points. Since the dimensions of the lane is known, I chose points on the ends of the line dashes. I then calculated the destination points of the warped top down perspective image from the lane dimensions. I assumed that the lane was 12 ft wide, the dash line was 10 ft long, and the space between dashed lines was 20 ft.

The colored points overlayed on the straight line test image represent the my chosen source points.

![alt text][image2]

The image below shows the above image after top down perspective transformation. Each source color point was mapped to its respective destination point in the warped image.

![alt text][image3]

#### 4. Lane Detection and Curve Fitting

Lane Detection uses a sliding window to determine peaks in the x direction of the binary warped image. A histogram of the all the detected points is created in the x direction. The function then centers the window at the histogram peak. The position of each window of the image is fitted to a 2nd order polynomial equation.

The image below visualizes this process with my curve_fit_plot() function. The green boxes represent the windows. The function starts and the bottom of the image and slides upwards. The window recenters itself based on the histogram peak.

![alt text][image6]

#### 5. Radius of Curvature.

I calculated Radius of Curvature in my set_curvature() function on the Line() class. The equation was given in the class notes. It uses the polynomial constants as inputs.

#### 6. Lane-Detected Image

The Lane-Detected Image below is the result of all the steps above. My process_ywx_bgr() function handles all of this. The lane is defined by the shaded green area bounded by the blue lane line on the left and a red lane line on the right. Radius of Curvature and Distance from Lane Center are shown in meters. Negative distance signifies the vehicle is left of the lane center. Positive distance means right of lane center.

![alt text][image7]

---

### Pipeline (video)

#### 7. Line() Class and Output Video

The Line() Class is required to smooth out lane finding in the video and validate the lane detection. It detects pixels from a binary warped image. It then applies a 2nd order polynomial curve fit to the detected pixels. 

Curves from multiple frames are saved in a buffer for smoothing. This makes detection more robust. Troublesome frames are discarded. A weighted average of this buffer is used to validate the detection of the line from the next frame. If the new line doesn't meet validation requirements, it is not added to the buffer.

Here's a [link to my video result](./project_video_process_full.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

My pipeline had difficulty on the lighter sections of the road. Here, the contrast is not as high as on the dark sections. This was mainly resolved by the validation algorithm of my Line() class. I used an exponentially weighted average for my buffer to weight recently detected curves more heavily. This allows the average to adapt to changing curves better.

I had carefully set each parameter to keep the line on the lane. If my validation was too strict, my algorithm would not follow the road as the curve changed. If my validation was too lenient, my algorithm would lock onto objects that were not lane lines.

I found that reducing the sliding window width prevented the lane detection from floating.

Since my pipeline mainly relies on color to detect lanes, it would likely fail if a white or yellow object fell into the lane detection window. Adding more sobel thresholds may help with this. Logic that filtered the shapes of detected points would improve robustness.

I'm not confident that my radius of curvature is correct. The class notes indicate that the curvature should be roughly 1000 m. My video shows curvature from 200-1000 m.