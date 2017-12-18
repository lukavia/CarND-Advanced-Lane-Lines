## Advanced Lane Finding Project

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

[image1]: ./images/undistorted1.png "Undistorted"
[image2]: ./output_images/undistorted/test3.jpg "Road Transformed"
[image3]: ./output_images/binary/test2.jpg "Binary Example"
[image4]: ./output_images/birdeye/straight_lines1.jpg  "Warp Example"
[image5]: ./images/lane_find_init.png "Fit Visual"
[image6]: ./output_images/test2.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  

This is the writeup ;)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./adv_lane_lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

In the second code cell I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  
In the next cell I create cal_undistort function that I use to undistort an image. I then applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

All other test images can be found undistorted in ./output_images/undistorted/

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image in the code cell where the function binary_pipeline is defined.  Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In the next code cell I define a class BirdEye that is used for perspective transform on the images. I chose a class since once the M matrix is found it can be reused on all images.  I chose the hardcode the source and destination points in the following manner:

```python
    src = np.float32([[268,676], \
                      [582,460], \
                      [705,460], \
                      [1043,676] \
                     ])
    dst = np.float32([[268,676], \
                      [268,0], \
                      [1043,0], \
                      [1043,676] \
                     ])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 268, 676       | 268, 676     | 
| 582, 460       | 268, 0       |
| 705, 460       | 1043, 0      |
| 1043, 676      | 1043, 676    |

I derived to those cordinates by taking the x, y cordinates of the left line from the file straight_lines1.jpg. And the right line from straight_lines2.jpg. I then preserv the base (bottom) point and straitend the top point to achive the same X position. 

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I then defined 2 functions: find_lanes_init and find_lanes. 

find_lanes_init takes the histogram for the bottom half of the image and use it the find a starting point for left and right lane finding. It then use a window from the find points to find the the next section of the line and if found more than 50 pixels it recenters the window for the next find. 
It looks like this:

![alt text][image5]

The find_lanes function uses the information from the previous find to make its findings. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the cell where I define the class Worker I use the following code to find the curveture of the lanes

```python
y_eval = np.max(ploty)
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])

```
I correct from pixel space to meters by assuming 3.7/700 pixels per meters for x
and 30/720 for y
I then take an avarage of the 2 lines to show on the image

I then find the diviation by the center of the image (where I assume the camera is mounted) and the center of the found lane base (y=720 bottom of the image).
```python
        offset = (img.shape[1]/2 - ((right_fitx[-1] - left_fitx[-1]) / 2 + left_fitx[-1])) * xm_per_pix
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of the final result

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There should be sanaty checks to ensure the road is found correctly. For example check to see if the lines are paralles more or less. Check the distance between the lines. 
I could alse take an avarage of the last X finds and check the results against the last finds. 
Once a problem is found worker.last_found should be set to false. 
Several failures and there sould be some emergency mode.   
