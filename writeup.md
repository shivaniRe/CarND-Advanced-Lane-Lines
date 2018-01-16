
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

[image1]: ./output_images/Undistortion_of_Image.png "Undistorted"
[image2]: ./output_images/Undistorted_Test_Image.png "Road Transformed"
[image3]: ./output_images/Combining_Thresholds_on_Grayscale_Image.png "Binary Example (Gray)"
[image4]: ./output_images/Combining_Thresholds_on_SChannel_Image.png "Binary Example (S-channel)"
[image5]: ./output_images/Combining_SChannel_and_Gradient_Thresholds.png "Binary Example"
[image6]: ./output_images/Perspective_Transform_of_Original Image.png "Warp Example"
[image7]: ./output_images/Perspective_Transform_of_Combined Image.png "Warp Example"
[image8]: ./output_images/VIsualization_of_sliding_windows.png "Sliding Window"
[image9]: ./output_images/Sliding_Window_Alternative.png "Fit Visual"
[image10]: ./output_images/example_output.png "Output"
[video1]: ./output_images/project_video_output.mp4 "Video"

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "Advanced_Lane_Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt text][image1]

### Pipeline (single images)

#### 1. Distortion correction of image.

To undistort a test image, I used the 'obj_points' and 'img_points' calculated in the previous step and applied them on the test image. The sample undistorted image looks like this one: 
![alt text][image2]

#### 2. Create a thresholded binary image. 

I tried different combinations of color and gradient thresholds. First I tried a combination of absolute threshold, magnitude of gradient and direction of gradient on a gray image. Here's an example of this step.
![alt text][image3]

Then I applied the same on s_channel image and here's an example of that (code for this is in code cells 5 and 6 of "Advanced_Lane_Lines.ipynb").
![alt text][image4]

In the end I combined color and gradient thresholds to generate a binary image (thresholding steps in code cell 7 of "Advanced_Lane_Lines.ipynb").  Here's an example of my output for this step.  
![alt text][image5]

#### 3. Perspective transform of an image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in code cell 8 lof "Advanced_Lane_Lines.ipynb".  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the source and destination points in the following manner:

```python
img_shape=(img.shape[1],img.shape[0])
src = np.float32(
    [[0,img_shape[1]],
    [img_shape[0]/2-120,img_shape[1]/2+100],
    [img_shape[0]/2+120,img_shape[1]/2+100],
    [img_shape[0],img_shape[1]]])
dst = np.float32(
    [[0,img_shape[1]],
    [0,0],
    [img_shape[0],0],
    [img_shape[0],img_shape[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 0, 720        | 0, 720        | 
| 520, 460      | 0, 0          |
| 760, 460      | 1280, 0       |
| 1280, 720     | 1280, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
![alt text][image6]

Below is an example of warped binary image.
![alt text][image7]

#### 4. Identifying lane-line pixels and fitting their positions with a polynomial

Once I generated the warped binary image, I took histogram along all the columns of the lower half of the image. The two most prominent peaks in this histogram are good indicators of the x-positions of the lane lines. I used that as a starting point for where to search for the lines. I then used a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.

Lastly, I used the x and y pixels positions to fit my lane lines with a 2nd order polynomial kinda like this:
![alt text][image8]
![alt text][image9]

#### 5. Calculation of the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated radius of curvature in pixel space in code cells 10 and 11 in "Advanced_Lane_Lines.ipynb". Then repeaed the calculation in meter scale in code cell 12.

#### 6. Result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 13 in `Advanced_Lane_Lines.ipynb` in the function `draw_lanes()`.  Here is an example of my result on a test image:
![alt text][image10]

---

### Pipeline (video)

Here's a [link to my video result](./output_images/project_video.mp4) and [link to my challenge video result](./output_images/challenge_video_output.mp4)

---

### Discussion

In this project, I first experimented with the threshold values when creating thresholded binary image. I also experimented with different channel immages and combining them with gray scale image. I decided to go with s-channel image with gray scale as it identified lanes better. My pipeline fails when there are shadows in the images. 
