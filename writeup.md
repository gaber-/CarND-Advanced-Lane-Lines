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

[image1]: ./saved/chessboard_corners.png "Chessboard corners"
[image2]: ./saved/road.png "Road Transformed"
[image3]: ./saved/chessboard_distorted.png "Chessboard distorted"
[image4]: ./saved/chessboard_undistorted.png "chessboard undistorted"
[image5]: ./saved/combined_threshold.png "chessboard undistorted"
[image6]: ./saved/warped.png "warped"
[image7]: ./saved/poly.png "poly"
[image8]: ./saved/lane.png "lane"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.

Example of chessboard with detected corners:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I applied the distortion correction to the test image using the `cv2.undistort()` function with the camera matrix and distortion coefficients computed with `cv2.calibrateCamera()`.

Since the distortion is barely visible on the test images I used a chessboard image to show the difference:

![alt text][image3] ![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.

![alt text][image5]

I used a combination of the gradient-related thresholds this way: I created some binary images by thresholding color or sobel gradients

I applied different thresholds to the same gradient map for the most important algorithms.

I summed all the binary images and applied a threshold to the resulting image, creating another binary image which is the result of the "voting" of the various binary maps. This held a better result than simply AND and OR-ing the binary images.

The Light part of HLS performs really well on white lines, but may encounter problems with other colors

Sobelx or sobel magnitude tend to sketch the contour of the lane lines, well but is prone to noise at high sensibility (low threshold)

Sobel gradient is really noisy but seems to catch the lanes in ways the other techniques I employed don't, and with a single vote it can't add noise to the image by itself.

The saturation part of HLS usually catches the lane even with different colors (eg blue, which other color-based tends to be missed) but is really noisy

The code can be found in cells 3 to 7.


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used a fixed set of points for the transform:

# Four source coordinates
src = np.float32([[545, 460],
                [735, 460],
                [1280, 700],
                [0, 700]])
# Four desired coordinates
dst = np.float32([[0, 0],
                 [1280, 0],
                 [1280, 720],
                 [0, 720]])

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

The code can be found in cells 10 to 12 (cells 7 and 8 contain project one's pipeline, which I used to identify the lines from which I extracted the points)


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified the pixels based on the aforementioned thresholds. I then used two techniques one for the first frame ond one for the following frames in each video:

The first one was based on a histogram, and I performed a window search looking for the most likely candidate area on each line, based on how concentrated the pixels were in the area.

On the second I selected the pixels within a given margin (100 in my case) and fit a different second degree polynomial to the selected pixels. The theory being that frames next to each other have similar polynomials

On this one I perform a sanity check: I assert that the distance of the car is less than 1.5 m away from the center of the lane, otherwise I perform the histogram based search again.

Here's an example of 2nd degree polynomial fit to the selected pixels:

![alt text][image7]

The first function is in cell 15 the second in cell 18


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of the curve using the computed polynomial. The results seem realistic.

The code is in cell 21

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I warped the detected lane with the opposite transformation as the first perspective transform, and applied it onto the original image. Example:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./out/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I started preparing the images, removing the camera deformation and the perspective deformation. I used some fixed points which yelded a good result.

Then I chose which algorithms to apply to generate a binary map of the interesting pixels. This is where I used saturation, color and gradient thresholding. The challenge here is how to find a good set among the wealth of techniques available (many of which redundant) in order to keep as many "good" pixels with as little noise as possible. This is complicated by the fact there are many different types of images whith different kinds of issues (you can't tune the thresholding parameters for a specific set of images because it wouldn't work on others).

Lastly I applied the techniques to extract the polinomials most likely to fit  the two lines and plotted them on the image.

The current implementation may be confused by sudden variations in curvature of the lane or orientation of the car and extreme curvatures of the lane.
