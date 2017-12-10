## Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objPoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgPoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.
Below is the image of the chessboard with corners found and lines drawn:

<img width="338" alt="chessboardcorners" src="https://user-images.githubusercontent.com/20146538/33803240-fdb7fb7a-dd3f-11e7-8009-37d9f5638062.png">

I then used the output `objPoints` and `imgPoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
Below is an example of original and undistorted image of a chessboard image

<img width="715" alt="distortion" src="https://user-images.githubusercontent.com/20146538/33803254-48341a4e-dd40-11e7-80e8-aea438438351.png">

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
<img width="649" alt="distortion_test" src="https://user-images.githubusercontent.com/20146538/33803267-b789f616-dd40-11e7-852d-c041306759ca.png">

After camera calibration using the `objPoints` and `imgPoints`, the image is undistorted using undistort function.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I visualized various threshold to generate the binary image ranging from absolute, magnitude, direction and various color space thresholds (thresholding steps at code cells 12 through 30)). Then, I chose the best parameters to get the combined threshold. In the combined threshold, I chose HLS and HSV color spaces. 

Here's an example of my output for this step.

<img width="912" alt="threshold" src="https://user-images.githubusercontent.com/20146538/33803310-b23f35b2-dd41-11e7-963f-938ab885bc32.png">

In this, I used the magnitude threshold on the s and l channels in HLS color space and h,s,v channels (as h and s detected the white and yellow lanes and v detected the yellow lane very pominently) in HSV color space. 
Below are the images showing the HLS and HSV color space channel outputs:
HLS color space:
<img width="797" alt="hls" src="https://user-images.githubusercontent.com/20146538/33803329-4856899c-dd42-11e7-9f1e-57fa8a5643a8.png">

HSV color space:
<img width="794" alt="hsv" src="https://user-images.githubusercontent.com/20146538/33803330-4b634292-dd42-11e7-9a77-6e8819d133ba.png">

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `image_warp()`, which appears in code cells 35 through 39.  The `image_warp()` function takes as inputs a binary image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([(500,450),
                  (780,450), 
                  (150,690), 
                  (1150,690)])
    dst = np.float32([(450,0),
                      (img.shape[1]-450,0),
                      (450,img.shape[0]),
                      (img.shape[1]-450,img.shape[0])])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 500, 450      | 450, 0        | 
| 780, 450      | 830, 0      |
| 150, 690     | 450, 720      |
| 1150, 690      | 830, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![warped](https://user-images.githubusercontent.com/20146538/33803448-4ee045e8-dd45-11e7-85f0-666c6c8e26c0.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I found the peaks using an histogram and then used 9 windows and then found the lanes by fitting my lane lines with a 2nd order polynomial using the code provided in lecture "Finding the lines" (code cells 40 through 42):

![lanes_detected](https://user-images.githubusercontent.com/20146538/33803462-b415f82c-dd45-11e7-8f38-303ee76ad556.png)

First, the peaks of the left and right halves of the histogram are found. These will be the starting point for the left and right lines respectively. Then, the pixels in the left and the right lane are found and then a 2 degree polynomial is fit to the lines.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cells 43 through 44. First a scale for pixel to meter conversion is made and then it is used to find the actual radius of curvature using the code provided in lecture "Measuring Curvature". I chose the midpoint to be the mid-point of the position where the left and right lanes start. I also calculate the deviation from the center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cells 48 through 49 in the function `lanes_on_image()`.  Here is an example of my result on a test image:

![lane_on_image](https://user-images.githubusercontent.com/20146538/33803547-c7da8664-dd47-11e7-90c9-05b5150daaf9.png)

---

### Pipeline (video)
Here's a [link to my video](https://drive.google.com/file/d/1QeglI44MkRbl88kDa6aobDjJ8YbiTrek/view?usp=sharing)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced an issue with using the line class as mentioned in the lecture "Tips and Tricks for the Project". I tried to get the exact video but I could get the proper output for highly curved lanes. It performs good but I wanted to get even better. Also, it really took a lot of time and effort to come up with the approprite threshold.