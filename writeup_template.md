
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


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/CameraCallibrate.png"> 
<img width="400"  src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/OriginalvsUndistorted.png"> 

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

<img width="400" src= "https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/OriginalvsUndistorted_v2.png">

In my code `undistorted_correction` converts the input image into grayscale image and the result is fed to camera calibration function. Next, `cv2.undistort` converts the input image to an undistorted image. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Several methods have been used in my code (please see cell 5) to identify a binary image. I resorted to several trial and errors to tune the parameters of thees functions.

<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/Original%26Sobelx.png"> 
<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/CombinedBinary.png"> 

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code consists of a function called `perspective_transform` which takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/WarpedvsUndist.png"> 

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

[Inspring from the trick presented in lecture](https://classroom.udacity.com/nanodegrees/nd013/parts/fbf77062-5703-404e-b60c-95b78b2f3f9e/modules/2b62a1c3-e151-4a0e-b6b6-e424fa46ceab/lessons/40ec78ee-fb7c-4b53-94a8-028c5c60b858/concepts/c41a4b6b-9e57-44e6-9df9-7e4e74a1a49a) I was abe to locate the lane lines and fit a Polynomial. The main idea here is identifying the peaks of two halves in the histogram output of. The function `find_lane_radius` is responsible to compute the left and right radii in my code.

<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/LaneBoundry.png"> 
<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/LaneBoundry_v2.png"> 
<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/TestPipeline.png"> 



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
I followed the simple role from math and calrified [here] in the lecture (https://classroom.udacity.com/nanodegrees/nd013/parts/fbf77062-5703-404e-b60c-95b78b2f3f9e/modules/2b62a1c3-e151-4a0e-b6b6-e424fa46ceab/lessons/40ec78ee-fb7c-4b53-94a8-028c5c60b858/concepts/2f928913-21f6-4611-9055-01744acc344f) to measure the radius of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

`my_pipline` method in `cell 14` is responsible to turn something similar to following image: 

<img width="400" src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_images/FinalResult.png">

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_video.mp4)

<img src="https://github.com/AliBaheri/Advanced_Lane_Lines_SDCND/blob/master/output_video.gif"/>

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The binary image is a key in this project. It is critical to detect lanes but not detecting other edges. Another important thing is, in general, hard-coding the perspective transform parameters is not a great idea. I am trying to find a better mechanism to tune those parameters. Furthermore, I observed that my pipeline is not much robust when shadows come into play. Again this question arises. Can we employ a mechanism to tune the parameters to design a much more robust pipeline?
