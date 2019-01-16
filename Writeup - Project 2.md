# Project 2 - Advanced Lane Finding Project

## Writeup
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

[image1]: ./output_images/output_6_0.png "Undistorted"
[image2]: ./output_images/lane_undistortion.png "Road Transformed"
[image3]: ./output_images/output_9_0.png "Binary Example"
[image4]: ./output_images/output_11_0.png "All Binary Example"
[image5]: ./output_images/output_14_0.png "Warp Example"
[image6]: ./output_images/output_18_1.png "histogram and sliding window"
[image7]: ./output_images/output_20_0.png "Quick search"
[image8]: ./output_images/output_26_2.png "Lanes on real image"
[video8]: ./lane_detection_output.mp4 "Video"

## Contents :
* The project is implemented in jupyter notebook, and comments are added at appropriate places.
* Hence, the code is self-explanatory, and important links are provided as a part of README.md
* Here, I shall explain how each and every [`rubric points`](https://review.udacity.com/#!/rubrics/571/view) are satisfied.

---

### Writeup / README


#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf.


* This file is the writeup, that contains explanation of how all other rubric points are satisfied

### Camera Calibration


#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.


* The code for this step is contained in the third code cell of the IPython notebook located in `Project 2 - Advanced Lane Detection.ipynb`, using the class `Camera_operations`. This is a singleton class, and the methods include
  * read_all_images
  * calibrate_camera
  * undistort_image
* This object is instantiated, fed with the sample chess board images, and test for distortion correction in **Fourth code cell**
* The `calibrate_camera`methods takes in number of columns and rows of intersection points of chess board images as arguments, works on the image list prepared by `read_all_images` method, and finds object points and image points for each and every eligible images. Using these points, camera calibration and distortion coefficients are deduced.
* Then, `undistort_image` method undistorts the images provided, using the camera matrix and distortion coefficients.

![alt text][image1]

### Pipeline (single images)


#### 1. Provide an example of a distortion-corrected image.


* One of the sample image is taken, and the above lane undistortion method is applied. This can be seen in the **Fifth code block**

![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.Provide an example of a binary image result.

* Sobel gradient masking and HLS masking is done, using separate api's, present in **Sixth code block**
* All the combinations of these filters are tested in **7th and 8th code blocks**

![alt text][image3]
---
#### Conclusion of above observation:
* The `h-channel` is of no use in finding the lane
* The `s-channel` works really good in identifying the lane lines
* The `angular` (a.k.a `direction`) binary finds the lane in road with darker shade, but also have too much noise
    - Hence, this can be combined with `magnitude binary` (using `bitwise-and`)
* The `x sobel` and `y sobel` works good with road with darker shade and finding white lanes, so do the magnitude binary
    * Hence, the `magnitude binary` can be used alongside `s-channel` (`bitwise-or`) 
* The `l-channel` works good in road with darker shade, but when the road shade changes, no lanes are detected (pixels of enitre road is activated)
    * Hence, this can be used as a **final filter**, after the `magnitude`, `direction` and `s-channel` filtering
---

* Based on the above observations, the `lane_filter` api is deduced, in **9Th code block**, and all the sample images are tested in **10th code block**.

![alt text][image4]




#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.


* The Singleton class has been implemented, that handles the perspective and inverse perspective transform, in **11th code block**
* Also, this singleton class is instantiated in **12th code block**, which also sets the source and destination points.
* Wrapper api's are implemented to acccess the singleton objects's `warp and unwarp` methods.
* Testing of this methods are done using sample images, in **13th Code block**

![alt text][image5]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?


* All the operations that deal with the lane line polynomial and curvature calculations are implemented as methods in one singleton class **Polynomial_lanes**, in **15th code block**
* The above rubric point is satisfied by the methods 
   * method `find_histogram` to deduce the histogram
   * method `sliding_window_search` to search for pixels that belong to lanes, using sliding window method
   * method `fit_polynomial` to find the polynomial equation, that matches the lane pixels identified
   * method `search_around_poly` (quick search) to find the lane pixels faster than sliding window search
   * method `check_sanity` checks whether the lane lines identified by quick search makes sense.
* All these methods are tested one after the other, the code block that follows.

![alt text][image6]
![alt text][image7]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.


* This requirement is satisfied by the following methods of **Polynomial_lanes** class:
   * methods `measure_curvature_real` and `find_offset`
   * method `measure_curvature_real` calculates the curvature radius using the formula:
   ```python
   # Calculation of R_curve (radius of curvature)
   left_curve_rad = ((1 + (2*left_fit_coeff[0]*y_eval*ym_per_pix + left_fit_coeff[1])**2)**1.5) / np.absolute(2*left_fit_coeff[0])
   right_curve_rad = ((1 + (2*right_fit_coeff[0]*y_eval*ym_per_pix + right_fit_coeff[1])**2)**1.5) / np.absolute(2*right_fit_coeff[0])

   ```
* We use the following ratio to convert from pixels to meters, to find curvature radius in real world mesurement
   ```python
   # These are arbitrary values mentioned in udacity classroom
        
        ############################################
        # In reality, this value depends upon the  #
        # co-ordinates that we choose for warping  #
        ############################################
        
        ym_per_pix = 30/720                  # meters per pixel in y dimension
        xm_per_pix = 3.7/700                 # meters per pixel in x dimension
        ploty = np.linspace(0, 719, num=720) # to cover same y-range as image
   ```
*  the method `find_offset` finds the offset of the car using the same conversion ratio.
   ```python
      center_of_lane = int((int(left_base) + int(right_base))/2)
        if center_of_lane > 640:
            side = "left"
            distance = ((center_of_lane-640)*(3.7/700))
        else:
            side = "right"
            distance = ((640-center_of_lane)*(3.7/700))
   ```


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

* This requirement is satisfied by the method `plot_on_real_frame` of class **Polynomial_lanes**
* The lanes are drwn on a warped binary image, unwarped, and drawn over original image using cv2.addWeighted

![alt text][image8]

## Pipeline (video)


#### 1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)

* The image processing pipeline that was established to find the lane lines in images successfully processes the video.
* Frames are searched for lanes using **sliding window search** , until the queue that holds that frame info becomes full (queue size = 10)
* After that, the new frames are searched for lane pixels using the polynomial of existing frame's lane `quick search`
* Sanity check is made and failure tolerence is defined, so if the quick search fails fit along with the previous frames, `sliding window search` is done again.

**As we can see in the final output, 363 frames are processed using sliding window search,
898 frames are processed using quick search. 
The entire process took 4 min 16 seconds to complete**

Here's a [link to my video result](./lane_detection_output.mp4).

## Discussion


#### 1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

* Problems Faced:
   * The organisation and grouping of all the operations, in a logical way was challenging.
   * Though many operations related to each other are grouped as classes, some remains still as api's
   * Choosing the co-ordinates for warping was hard, as the coordinated decided for one straight line image, didn't fit for warping of image with curved lanes. The lanes went out the warped image in some scenarios.
  
* Failure Cases :
   * As the filtering thresholds and technique are deduced manually by trial and error, this resulting overall filter is not robust.
   * The Filter might fail, when processing another video.
   * The co-ordinates used for warping is ddeduced manually. This introduces inconsistency in :
      * Finding the radius of curvature
      * Processing new video streams
   * Using the sanity check based on radius of curvature is not completely accurate:
      * As the road becomes narrow, the curvature radius increases exponentially
      * Comparing it with average curvature radius of previous curved frames does not make sense.
      * But this check holds good in tracking the curved lanes.
      
* Possible Improvements:
   * A better solution to for dynamically finding the co-ordinates for warping
   * Curvature of the lane for narrow roads should not be considered, as they are incredibly high, and varies fast.

* A general and robust pipeline should be built, that handles the challenge video too.
