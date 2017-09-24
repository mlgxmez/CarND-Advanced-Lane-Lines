## Advanced Lane Finding Project
To clarify, this project has been divided into two Jupyter notebooks. The first one `Lane_Detection.ipynb` was to evaluate the techniques of image distortion correction and post-processing for a test image. On the second notebook `Advance_Lane_Pipeline.ipynb`, an end-to-end solution was developed to detect the lane from a video sequence. A good example to industrialize an application for mass production.

---

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

[image1]: ./report_images/distortion_chessboard.png "Undistorted"
[image2]: ./report_images/distortion_road.png "Road Undistorted"
[image3]: ./report_images/perspective.png "Perspective transformation"
[image4]: ./report_images/perspectiveHLS.png "Perspective transformation and separation by channels"
[image5]: ./report_images/perspectivebinary.png "Perspective transformation and binarization"
[image6]: ./report_images/videoframe.png "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd and 4th code cells of the first notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

This distortion correction was then applied to the images obtained from the mounted camera of the car.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

It was better to apply a perspective transform before image thresholding because the focus was to accurately detect the lines of the lane. The rest of the image has no valuable information for the objective of the project.

Since the perspective of the road from the camera doesn't change at all, we defined a set of source and destination points as arguments for the `cv2.getPerspectiveTransform()` function in order to get the perspective transform and its inverse. These points are defined as follows:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 320, 0        | 
| 205, 720      | 320, 720      |
| 1110, 720     | 960, 720      |
| 703, 460      | 960, 0        |

Then we test the transformation on a real image and we obtained a top-down view of the road. This contained on the 7th cell of the first notebook. The result is shown below

![alt text][image3]

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a color threshold to generate a binary image. I made the conversion to the HLS color space and set manually the threshold values for the L and S channels in order to extract the lines. As you can see the L channel is used to get the right line whereas the S cahnnel is used to get the left line. This section is contained in the 8th and 9th cell of the first notebook and the results can be shown below these lines. 

![alt text][image4]

After setting the threshold values:

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Everything from now on will appear on the second notebook. On it, we encapsulated the function to find the lines, fit a polynomial, get the radius of curvature and paint the valid region of the line. Since this code is for production, all the intermediate steps doesn't show a visual evidence and just the final output is shown (the video with the vehicle postion and radius of curvature).

* The strategy followed for finding lines was to define the histogram on the binarized warped image and find the region where the accumulation of white pixels is the highest. Then, we define a sliding window moving upwards to find those white pixels in chunks.
* We extract the pixels' position on the binarized image and fit a polynomial. In case no pixels are detected for the current image, we compute the average of the fitted x of the last 5 frames. And we use the result as the best fit of the polynomial. This has an impact when the car crosses a section of the lane with a different look than in the previous frames. Nonetheless, the algorithm is capable of maintaining the lane region painted all the time.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

* The radius of curvature is calculated on the function `get_rad_curvature()` function, and the lane position is calculated on the function `paint_lane()`. Both appear on the second notebook. The curvature was coded following the guidelines of the project. For the lane position, the offset between the center of the lane and the center of the image (that can be translated as the position of the car). This offset is then transformed in meters and this info is shown on the video  

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the function `paint_lane()` we recover the original image together with lane region painted over it.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The suggestion of using a class instance to keep data of the lines frame after frame was a good idea. However, the class is adapted to your problem and not all the methods listed on the help where not really needed. The approach presented in this project struggle with patchy section of the lane and the curvature changes smoothly. The algorithm can severely suffer in videos showing a crossroad a very closed curvature and even road where dummy lanes are painted because of road work.
