# **Finding Lane Lines on the Road** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

## Overview

When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project, lane lines detection is done in images using Python and OpenCV. OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, here is how the grayscale image looks:

![gray_scale](https://user-images.githubusercontent.com/20146538/30404133-ea4f6e82-9899-11e7-825c-66303ff8f960.png)

Then, I applied Gaussian smoothing on it with kernel size 5, which helps to remove the noisy parts of the images which that gives more reliable outputs in next stage. The output looks like this:
![gaussian](https://user-images.githubusercontent.com/20146538/30404180-3ed51b78-989a-11e7-8b9f-a4b1f0361a46.png)

Then, to find the image edges, I using canny algorithm. Select its parameters to find the required edges (strength of the edges to be identified). The output of canny function is as follows:
![canny](https://user-images.githubusercontent.com/20146538/30404359-3ba8090a-989b-11e7-9e2c-b6b3706b4ff9.png)

After finding the edges, I defined the polygon region of interest so that it only includes the region which will actually be in the scope of the car camera and then masked the left out region. 
![masked](https://user-images.githubusercontent.com/20146538/30404427-7f6c8eb8-989b-11e7-886c-f10d7c823567.png)

After this, I used Hough transform to find the lanes from canny edges. For this, I set the parameters of the hough transform function to include the points which form a line. The image after hough transform is as follows:
![hough_ini](https://user-images.githubusercontent.com/20146538/30404595-3e3565cc-989c-11e7-9bc9-62abfd797121.png)

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by taking the slope between the two points to be joined and then checking whether the slope represents the points in left or right line. The left and right points are stored in separate lists. Then cv2 fitline function is used to fit the line into these points and then slope and intercept is calculated. This is done for multiple frames and then average value of left and right slopes and intercepts is calculated. Using these values, the end-point coordinates of the extended lines are calculated and the minimum value of the y-dimension of the region mask (320 in my case). Then, I used these coordinates to draw the left and right line.

![masked_lined](https://user-images.githubusercontent.com/20146538/30404837-7ffc434e-989d-11e7-976b-f82669b496c2.png)

After drawing the line on the masked image, I drew the line on the original image.
![final](https://user-images.githubusercontent.com/20146538/30404872-aeccd076-989d-11e7-8d5c-064fc8bd7087.png)

### 2. Identify potential shortcomings with your current pipeline
The shortcoming which I can see as of now is that my algorithm doesn't work for the challenge video. It runns into an error of cannot convert NaN float into integer.
Secondly, region selection is static

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to tune my algorithm to suit any type of input. I am working on removing the unaccepted lines and then choosing only the ones who's slopes match my requirement. I have tried removing the lines with slopes less than 0.5. 
Another improvement can be making region selection more dynamic.
