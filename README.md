# **Lane Finding**

## Computer Vision to Build a Lane Finding Algorithm

### Using a images/video via a cameara as the input

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

The code is located in Ipthon Notebook located in "./Advanced_Lane_Finding.ipynb"

[//]: # (Image References)

[image1]: ./camera_cal/calibration3.jpg "Chessboard Image"
[image2]: ./camera_cal/corners_found15.jpg "Drawn Chessboard Image"
[image3]: ./output_images/undistortedchess.jpg "Undistorted Chessboard Image"

[image4]: ./test_images/straight_lines1.jpg "Test Image"
[image5]: ./output_images/1.undistorted.jpg "Undistorted"
[image6]: ./output_images/2.threshold_bin.jpg "Thresholded Binary"
[image7]: ./output_images/3.warped.jpg "Warped"
[image8]: ./output_images/4.fit_polynomial.jpg "Fit Polynomial"
[image9]: ./output_images/5.search_from_previous_frame.jpg "Search from Previous Polynomial"
[image10]: ./output_images/6.lane_detected.jpg "Lane Detected"
[image11]: ./output_images/7.final.jpg "Final Output"
[image12]: ./output_images/index.png "Before After"


[video1]: ./project_video_output.mp4 "Video"
[video2]: ./challenge_video_output.mp4 "Challenge Video"
[video3]: ./harder_challenge_video_output.mp4 "Harder Challenge Video"

---
### README

- The before and after of the algorithm successfuly drawing on detected lane as seen below and here is a link to my [project code](https://github.com/ianleongg/Advance-Lane-Finding/blob/master/Advanced_Lane_Finding.ipynb).

![alt text][image12]

### Steps used to achieve lane detection.

#### 1.Compute the camera calibration matrix and distrotion coefficients given a set of chessboard images.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The chessboard image and drawn chessboard image can be seen as following.

![alt text][image1]
![alt text][image2]


#### 2. Apply a distortion correction to raw images.

I applied this distortion correction to both the chessboard image and test image using the `cv2.undistort()` function and obtained this result: 

**BEFORE:**
![alt text][image1]
**AFTER:**
![alt text][image3]
**BEFORE:**
![alt text][image4]
**AFTER:**
![alt text][image5]


#### 3. Use color transforms, gradients, etc., to create a thresholded binary image.

I used a combination of color and gradient thresholds to generate a binary image. The color threshold I used was using the HLS channel where Hue channel helps detecting the lines thats independant of change in brightness and Saturation channel helps measure the colorfullness. The gradient thresholds I used included the applying x,y sobel, magnitude and direction of the gradient. After various hyperparameter tunings, the returned thresholded binary image is as following:

![alt text][image6]


#### 4. Apply a perspective transform to rectify binary image ("birds-eye view").

First, I did a trial and error to determine the src and dst points for the transformation. 

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 588, 470      | 320, 0        | 
| 245, 719      | 320, 720      |
| 1142, 719     | 960, 720      |
| 734, 470      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image as seen:

![alt text][image7]


#### 5. Detect lane pixels and fit to find the lane boundary.

I first performed a lane finding method by finding peaks in a histogram. I grabbed only the bottom half of the image as the lane lines are likely to be mostly vertical nearest to the car. The two peaks detected are used as starting points. Then I used a sliding window method and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image8]

To have a more robust model, I included another 2nd order polynomial fitting function which uses the values from the previous frame as lane lines don't necessarily move a lot from fram to frame. A blind search using the sliding window method can be reduced and instead just search in a margin around the previous lane line position:

![alt text][image9]


#### 6. Determine the curvature of the lane and vehicle position with respect to center.

I first defined the conversions in x and y from pixels space to meters such as:

ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension

A calculation of R_ curve and offset of car assuminng camera is mounted at car centerline can be calculated to return the left/right curvature radius and offset.


#### 7. Warp the detected lane boundaries back onto the original image.

I once again applied a perspective transformation back to the original image with the detected lane boundaries.

![alt text][image10]

#### 8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

With the calculated measurements from step 6., we added a display of the measurements onto the original image. 

![alt text][image11]


### Pipeline

A function defined as`def pipeline(img)` uses the image as an input and performs the steps mentioned above to output a lane detection image.


#### 1. Project video

The pipeline is functioning as expected in the video.

Here's a [link to my video result](./project_video_output.mp4)


#### 2. Challenge video

The pipeline is still acceptable even with the half the lane that is newly tarred. However, there are frames where the lane detected overlaps the half of the lane to the right as a black car drivesby.

Here's a [link to my video result](./challenge_video_output.mp4)


#### 3. Harder Challenge video

The pipeline starts to break when the road turnings are more extreme. It still detects the lane but with much lack accuracy when the road is winding.

Here's a [link to my video result](./harder_challenge_video_output.mp4)

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Overall, it was a great learning experience to understand and implement a car lane detection algorithm. Tweaking around the color and gradient threshold to produced a threshold binary image was tricky as there were alot of trial and error dispite first analyzing the threshold needed. However, the thressholded image step was crucial in lane detection for the video, challenge, and harder challenge video. A sliding window method was implemented to fit a second order polynomial but another method was implemented by using the results for sliding window method to reduce restarting the finding lane pixel step from sratch.

The pipeline will likely fail when theres a white/black car approaching near the camera region and when the road gets too winding. A better thresholded algorithm and better polynomial fitting algorithm can be implemented to further enhance pipeline results.
