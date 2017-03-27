##Writeup Template
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

[image1]: img/cal.png "Calibration 1"
[image2]: img/und1.png "Undistortion 1"
[image3]: img/und2.png "Undistortion 2"
[image4]: img/und3.png "Undistortion 3"
[image5]: img/thre1.png "Thresolds 1"
[image6]: img/thre2.png "Thresolds 2"
[image7]: img/thre3.png "Thresolds 3"
[image8]: img/warp.png "Warped image attempt 2"
[image9]: img/warp1.png "Warped 1"
[image10]: img/warp2.png "Warped 2"
[image11]: img/warp3.png "Warped 3"
[image12]: img/hist1.png "Histogram 1"
[image13]: img/hist2.png "Histogram 2"
[image14]: img/hist3.png "Histogram 3"
[image15]: img/lane1.png "Lane finding 1"
[image16]: img/lane2.png "Lane finding 2"
[image17]: img/lane3.png "Lane finding 3"
[image18]: img/lanes1.png "Lanes in image 1"
[image19]: img/lanes2.png "Lanes in image 2"
[image20]: img/lanes3.png "Lanes in image 3"
[video1]: video/project_video_out.mp4 "Video"
[image21]: img/cal2.png "Calibration 2"
[image22]: img/cal3.png "Calibration 3"
[image23]: img/warped1.png "Warped image attempt 1"
[image24]: img/warped3.png "Warped image attempt 3"
[image25]: img/warped4.png "Warped image attempt 4"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/JosuVicente/CarND-Advanced-Lane-Lines/edit/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

Note that I use two notebooks. "./project/P4 - final.ipynb" has the code for the project and "./project/P4 - visualizations.ipynb" has similar code but it includes visualizations of each step. When I explain each step it will relate to the first notebook but I´ll include images from the second one.

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for the calibration is contained in the second code cell of the IPython notebook located in "./project/P4 - final.ipynb". The code for the undistortion is located on third cell of the same notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test images using the `cv2.undistort()` function and obtained these results: 

![alt text][image1]
![alt text][image21]
![alt text][image22]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will show three examples of images and how they look after applying undistortion:

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. This is done on cell 4 of the IPython notebook located in "./project/P4 - final.ipynb". I created a list of functions that make use of different kind of thersolding like sobel, magnitud and direction of gradient and also for the HLS color space. At the end I have a function caalled thresold_pipeline that make use of the other functions to get a combined binary image like the ones below:

![alt text][image5]
![alt text][image6]
![alt text][image7]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears in on cell 5 of the IPython notebook located in "./project/P4 - final.ipynb".  The `unwarp()` function takes as inputs an image (`img`) and returns a warped image. Source (`src`) and destination (`dst`) points are calculated inside the function and I used different approaches ended up using at the end the second attempt from the images below:

![alt text][image23]
![alt text][image8]
![alt text][image24]
![alt text][image25]

I chose the hardcode the source and destination points in the following manner:
```
src = np.float32([
            [  598,   448 ],
            [  685,   448 ],
            [ 1055,   677 ],
            [  252 ,   677 ]])
dst = np.float32([[offset, 0], 
                  [img_size[0] - offset, 0], 
                  [img_size[0] - offset, img_size[1]], 
                  [offset, img_size[1]]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 598, 446      | 250, 0        | 
| 691, 446      | 1030, 0      |
| 1126, 673     | 1030, 720      |
| 153, 673      | 250, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto some tests images and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image9]
![alt text][image10]
![alt text][image11]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

On cell 6 of the IPython notebook located in "./project/P4 - final.ipynb" I created the function `preprocess()` which will receive an image and return and it will go though all the process explained so far returning at the end and unwarped image.

This unwarped image is then processed in cell 7 of the IPython notebook located in "./project/P4 - final.ipynb" on the function `findlines()` that also accepts previous left and right lane fits so it doesn´t have to use sliding windows to search where the lane lines are although I ended up using them for every image.

On this process one of the first thing is creating and histogram to then look for the lanes around the area were there are peaks. An example of some of those histograms can be seen here:

![alt text][image12]
![alt text][image13]
![alt text][image14]

And after applying the sliding windows method and fitting a 2nd order polynomial it loos like this:
![alt text][image15]
![alt text][image16]
![alt text][image17]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell 8 of the IPython notebook located in "./project/P4 - final.ipynb" and I also calculate the offset. It´s all done inside function `get_curvature_and_offset()` and the results are displayed in the final video.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 9 of the IPython notebook located in "./project/P4 - final.ipynb" inside `process_image()` function. Here are some examples of my result on tests image:

![alt text][image18]
![alt text][image19]
![alt text][image20]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://raw.githubusercontent.com/JosuVicente/CarND-Advanced-Lane-Lines/master/video/project_video_out.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

