# Advanced Lane Finiding - CarND - Project 4

## Camera Clabration

For this step the 20 chessboard images provided in the folder camera_cal are used. All the images are read and the 9x6 corners are identified using the function findChessboardcorners available in OpenCV. If the corners are found, the image points and object points arrays are appended with new information. After processing all the chessboard images the final image points and object points arrays are used to calculate the camera calibration matrix using calibrateCamera function in OpenCV. This matrix is used throughout the project to undistort the images. Using the calibration matrix all the chessboard images are corrected for distortion. An example is given below.

![](https://github.com/pratvdev/CarND-AdvancedLaneDetection/blob/master/output_images/Ori-Undist2.png?raw=true)

## Test Images Pipeline

All the functions required to process the images are defined first in the python notebook. Below is the description of different pipeline steps and examples.

### 1) Distortion Correction

For this step the calibration matrix calculated from chessboard images is used. All the images are processed using the *cal_undistort()* function. An example for an undistorted test image is given below.

![](https://github.com/pratvdev/CarND-AdvancedLaneDetection/blob/master/output_images/Ori-Undist_test_images_2.png?raw=true)

### 2) Warping/Perspective Transform

After undistorting the images, four vertices are defined in a genral way so that they include the current driving lane path. These are the source vertices which will be used to transform the lane path. The transformed image shows the birds eye view of the lane path. Four vertices which represent a perfect rectange are chosen as destination vertices. The transformed lane path is shown inside these four vertices on image. The function *warp()* is used to transfrom the image. The *warp()* function produces a warped image and also gives inverse transform matrix as outputs. The inverse transform matrix is used later in the pipeline to plot the lane path in its original form. An example image is given below.

![](https://github.com/pratvdev/CarND-AdvancedLaneDetection/blob/master/output_images/Warped_Example.png?raw=true)

### 3) Finding Yellow and White Lane Markings

The next step involves identifying the yellow and white lane markings on the transformed image. Lane markings are mostly yellow or white in color on dark backgrounds. To identify these lane markings the RGB image is converted in HSV color spectrum using OpenCV. Masks for yellow and white pixels on the image are created. This mask creating involves defining the miminum and maximum threshold values for white and yellow colors in the HSV spectrum. To perform this step the function *find_Yellow_White_Lanes()* is used. This function provides an image which contains a combination of both white and yellow colored pixels within the defined thresholds.

### 4) Applying Sobel Filter

The transformed image is passed to a function *abs_sobel_thresh()*. This function applies a Sobel filter along the X direction on the transformed image and absolute value is calculated. The resulting image is filtered for threshold values of min = 20 and a max = 150. The kernel size of 3 is chosen for this operation on the transformed image.

### 5) Combining Images

To identify the lane markings, the image outputs from step 3 and step 4 are combines. Step 3 gives us a masked images which contains yellow and white lane markings and step 4 gives us an image with threshold sobel filter applied along x direction. The pixel values greater than or equal to *0.5* from step 3 or pixel values equal to *1* from step 4 are considered for the combined image. An example is given below.

![](https://github.com/pratvdev/CarND-AdvancedLaneDetection/blob/master/output_images/Combined_Example1.png?raw=true)

### 6) Finding Lane Marking Mathematically

The combined image is then used to identify the lane markings and generate polynomials for the lane lines. To identify the dense parts(lane lines) a histogram is plotted along the x axis starting at the bottom. The histogram peaks show where the majority of the pixels are located. This is shown by the two prominent peaks onn the plot as the image is binary. Sliding window concept is used to identify the lane lines after this. The left and right prominent peaks are identified from the histogram and these will serve as the starting points for the sliding window. A total of *9* sliding windows have been chosen to identify the lane lines. To identify the lane minimu number of pixels need to identified inside the window. After finding such location the left and right lane indices are updated. The x and y coordinates for the lines are extracted from the indices and a second order polynomial is fitted to the lanes using *numpy.polyfit* function. 

For this whole process the function *findLanes()* is used. The calculated x and y coordinates are outputs for this function and are later used to draw the lane path polygon on the original image. An example image for detected lane lines is shown below.

![](https://github.com/pratvdev/CarND-AdvancedLaneDetection/blob/master/output_images/lane_fit_example.png?raw=true)

### 7) Drawing Lane Path

The next step uses the left and right lane polynomials from *findLanes()* to draw the actual lane path on the original image. For this the image needs to be inverse trasformed. For this the *Minv* matrix calculated from step 2 is used.  After inverse transforming the left and right lane points are calculated. These calculated lane points are used with *fillPoly()* function from OpenCV to draw the polygon on the image. This whole process is performed in the function *drawLane()*.

### 8) Calculating Radii of Curvature and Vehicle Position

To calculate the radius of curvature for left and right lanes, the output x and y coordinates calculated from step 6 are used. The image contains pixels and does not show the distance in standard units. x and y distances in pixels are converted into meters using the conversion factor *(30/720)*(lane is 30 meters long) in y direction and *(3.7/1000)* (lane width is 3.7 meters) in x direction. The lines polynomials for the lanes are calculated using the world coordinates and the radii of curvature are calculated. For calculating radii of curvature the function *radCurvature()* is used.

The camera for the images/video is assumed to be at the center of the car and its position relative to the center of the lane is calculated. The function *getVehPos()* is used to calculate vehicle position in this project. The mean of all the left lane x coordinates and the mean of all the right lane x coordinates are calculated. As the camera is assumed to be located at the center of the image, half of the total number of pixels on the x axis give the center. This value is subtracted from the mean of the left and right lane x value means. The resulting position value will be in pixels. the value *3.7* needs to be divided with the position value in pixels inorder to get vehicle position in meters.


### 9) Final Output Image

After calculating the radii of curvature and vehicle position, these values are written on the output image using the function *putText()* available from OpenCV.

An example image is given below.

![](https://github.com/pratvdev/CarND-AdvancedLaneDetection/blob/master/output_images/Final_Image_Example.png?raw=true)


*Note: More example images are available in the folder Output_Images*

## Video Pipeline

To process the video the moviepy libraries are imported. The input video is read and the pipeline process defined above is performed on every frame of the video and an output video is generated.

The output video for *project_video.mp4* is *project_output.mp4*. This output video is available in the repository.

The output video for *challenge_video.mp4* is *challenge_output.mp4*. This output video is available in the repository.


## Discusstion

Initially the combination of HLS image along S and the directional threshold filter were chosen for the transformed images for step 5. But after using that combination on the video the results were not satisfactory even though the combination worked for the test images. After some tuning of the parameters and researching on the internet, the combination choices for the image are changed to identifying yellow and white lane markings and sobel filter output. 

This combination worked well for the test images and the project video, but failed badly for the challenge video. The challenge video has different parts of the road in different colors which is not detected well by the chosn combination. Further tweaking of filter parameters or trying another combination might work during scenarios presented in challenge video. This needs to be explored further.

