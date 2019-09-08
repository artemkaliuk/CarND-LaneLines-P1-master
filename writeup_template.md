# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_images/hsv.jpg "HSV image"
[image2]: ./writeup_images/roi.jpg "ROI with canny edges"
[image3]: ./writeup_images/hough.jpg "Extrapolated lane markings based on Hough lines"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.
The pipeline can be generally described by following steps/procedures:
1) Color mapping and thresholding. Here, first an RGB to HSV mapping has been performed in order to deal with the lighting conditions in the "challenge" task. In the next step, yellow color thresholding has been applied in order for the yellow lines to stand out. In parallel, a grayscale transormation and, subsequently, white color thresholding have been applied on the original RGB image. The resulting two masks ("yellow" and "white") have been combined by an bitwise OR operator. The resulting mask has then been applied to the grayscale version of the original image, resulting in blackened pixels, which do not fit into the engineered yellow-white palitre.

![alt text][image1]

2) Gaussian blurring of a kernel size 7 has been applied to a color-reduced image in order to get rid of some high-frequency residual effects.

3) Canny edge detection filter has been applied to a smoothened image with following parameters:

    - canny_low_threshold = 40
    - canny_high_threshold = 50
    
This set of parameters allowed for a better detection of dashed lane markings which, in turn, led to a stabler line detection and extrapolation.
   
4) An fixed region of interest has been set up in order to neglect the side effects (e.g. the white car on the neighboring lane in the first sequence) and to concentrate on the ego-lane relevant edges. As the "challenge" sequence was captured by a different camera, the region of interest is not fixed on specific pixel values, but is defined in proportion to the image size.

![alt text][image2]

5) Finally, a block including Hough transform and extrapolation was applied. Hough transformation was used with the following parameters:
    - rho:1
    - theta: pi/180
    - min_line_length = 20
    - max_line_gap = 100
    
Due to a specific nature of the dashed lane markings in these sequences , it was important to use a liberal value for the minimum line length. As the space between the dashed markings is huge, a high number of pixels was used to define the maximum line gap.
The chosen extrapolation technique was fairly simple: for each Hough line a slope and intercept value was calculated. Based on the sign of the slope (negative for the left and positive for the right), all the Hough lines were assinged to a left line or a right line pool. An additional safeguard measure was applied in order to ignore hough lines with slope values out of the desired functional range. For each of the line pools, average values for slope and intercept were calculated. Then, in order to extrapolate the detected lane markings, the ROI vertices' y-coordinates were taken as anchor points. Based on their coordinates and on the calculated average values for the slope and intercept, the corresponding x-coordinates were found. In the end, line composition was performed based on the x- and y-coordinates using the cv2.line() function.
 
![alt text][image3]



### 2. Identify potential shortcomings with your current pipeline

1) As it can already be seen in the tested video sequences, linear fit will perform suboptimally on curvy roads (with both horizontal and vertical curvatures). 
2) A static ROI will cause troubles if any obstacles intrude in this region (e.g. lane change to the ego-lane by one of the vehicles in the neighboring lanes).
3) Lateral control with the current implementation might prove to be unstable due to the high-frequency noise in the slope and intercept values.

### 3. Suggest possible improvements to your pipeline
Recommended improvements address the above mentioned shortcomings.
1) A polynomial fit will be a better solution for curvy roads. Usually, a 3rd grade polynomial will be a good fit. However, detecting vertical curvatures is a non-trivial task.
2) A dynamic ROI can be employed by extracting the pixels occupied by detected objects (e.g. vehicles) from the predefined ROI area. Additionaly, semantic segmentation can be used in order to identify the image sections associated with the road surface. After all, for a full-self driving technology detection of neighboring lanes shall also be employed (lane change functionality) - thus, a dynamically enhanced ROI is a must.
3) Temporal filtering (e.g., mean or even Kalman) can be used in order to stabilize the detected lane markings. Alternatively, the lateral control issue can also be solved within the trajectory planning module (though the mechanism will be similar to the recommended one). Another synthetic measure can be used by manually shrinking the ROI (setting a higher pixel value for the ROI top y-coordinates). In terms of a function, this will mean reducing the reaction time by setting the target point for the lateral control closer to the ego vehicle. While this measure might stabilize the lane marking detection (due to the better resolution in the near range), this might cause oscillation effects in the control loop, especially if driving at higher speed.