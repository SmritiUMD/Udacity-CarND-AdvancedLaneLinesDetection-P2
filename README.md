# Udacity-CarND-AdvancedLaneLinesDetection-P2

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

![alt txt]( ./https://github.com/SmritiUMD/Udacity-CarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step2.png)
[image2]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step3-1-1.png
[image3]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step3-1-2.png
[image4]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step3-2.png
[image7]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step3-3.png
[image8]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step3-4.png
[image5]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step3-5.png

[image9]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step4.png
[image10]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step5-1.png
[image11]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step5-2.png
[image12]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step5-3.png
[image13]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step7.png
[image14]: ./https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/tree/master/output_images/step8.png
[video1]: ./project_video.mp4 "Video"



### Camera Calibration

#### In Step 1, I did the Camera Caliberation.

The code for this step is contained in the first code cell of the Python notebook.
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.   

### Pipeline (single images)

#### 1. Distortion-corrected image.

OpenCv funtion, cv2.undistort, will be used to undistort images.
![alt text][image1]

#### 2. In the Step3 I applied various transformations on image to generate binary image.

#### A. Calculation of directional gradient by sobel_x and sobel_y.
I applied sobel_x and sobel_y on the image and took absolute values.
![alt text][image2]
![alt text][image3]
#### B. Calculation of gradient magnitude
 I applied sobel_x and sobel_y on the image and then found the magnitude by sqrt(sobel_x**2+ sobel**2).
 ![alt text][image4]
#### C. Calculation of direction of gradient
I applied sobel_x and sobel_y then calculated gradient by np.arctan2(np.absolute(sobely), np.absolute(sobelx)).
![alt text][image7]
#### D. Calculation of color threshold
I used S channel because it gives best result for this purpose.
![alt text][image8]

#### E. Combining thresholds

I combined all the above steps to produce a binary thresholded image.

 Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image5]

#### 3. In Step4, Application of Perspective transform to get bird eye view.Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`,  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python

    if src_coordinates is None:
        src_coordinates = np.float32(
            [[280,  700],  # Bottom left
             [595,  460],  # Top left
             [725,  460],  # Top right
             [1125, 700]]) # Bottom right
        
    if dst_coordinates is None:
        dst_coordinates = np.float32(
            [[250,  720],  # Bottom left
             [250,    0],  # Top left
             [1065,   0],  # Top right
             [1065, 720]]) # Bottom right   
             
  ```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 280,  700     |250,  720      |
| 595,  460     |250,    0      |
| 725,  460     |1065,   0      |
| 1125, 700     |1065, 720      |
             
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image9]

#### 4. In Step5,  Histogram plotting. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

#### A. Detection of lanes for first frame.

I plot histogram of bottom half of image to detect lane lines. In the function detect_lines, I implemented the code for drawing windows by selecting starting points as histogram peaks for left and right lanes, then slided the windows by updating the starting points. I looped over nwindows, with the given window sliding left or right if it finds the mean position of activated pixels within the window to have shifted.Once I know the boundaries of our window, I found out which activated pixels from nonzeroy and nonzerox above actually fall into the window and recenter upon exceeding minpix(predefined).I used cv2.polyfit to fit a polynomial for left and write lanes.
![alt text][image10]
![alt text][image11]

#### B. Detection of Lanes for Upcoming frames
By searching in a margin around the previous lane line positio. The green shaded area shows where I searched for the lines this time. So, once we know where the lines are in one frame of video, I took the polynomial functions I fit before (left_fit and right_fit), along with a hyperparameter margin, to determine which activated pixels fall into the green shaded areas for a highly targeted search for them in the next frame.I fit a polynomial to all the relevant pixels you've found in your sliding windows by fit_poly() and set the area to search for activated pixels based on margin out from fit polynomial within search_around_poly.

![alt text][image12]

#### 5. In the Step6, Finding Curvature of Lines. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the function radius_curvature, I implemented the code to find radius of curvature by the polynomial. 

#### 6. In Step7, Warping the detected lines back to image. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented the code in function draw_lanes to warp the detected lines back.

![alt text][image13]
![alt text][image14]

---

![alt txt](https://github.com/SmritiUMD/Udacity-Self-DrivingCarND-AdvancedLaneLinesDetection-P2/blob/master/Lanedetection.gif)
