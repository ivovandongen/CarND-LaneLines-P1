# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./assets/gs_solidYellowCurve.jpg "Grayscale"
[image2]: ./assets/blur_solidYellowCurve.jpg "Blur"
[image3]: ./assets/canny_solidYellowCurve.jpg "Canny"
[image4]: ./assets/masked_solidYellowCurve.jpg "Masked"
[image5]: ./assets/hough_solidYellowCurve.jpg "Hough"
[image6]: ./assets/extrapolated_solidYellowCurve.jpg "Extrapolated lines"
[image7]: ./assets/final_solidYellowCurve.jpg "Final result"

---

### Reflection

### 1. Description of the pipeline

My pipeline consisted of 7 steps. First, I converted the images to grayscale, then I blurred the images with a Gaussian blur. 
Next a canny transformation is applied to extract the edges. 
Then a hough transform is applied to get line definitions from the Canny edges. 
The lines are then divided in two sets for the left and the right lane, averaged and extrapolated and this is finally overlayed on the original image.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function 
by first calculating the slope, y-intercept and length of each line. Then the the lines that have a 
reasonable slope (are in the general direction of the road) are divided in two sets, one for the left 
and one for the right lane.
The sets of lines are then averaged, applying the length of the line as a weight to ensure the longer lines in the near
field contribute most to the average direction and y-intercept of the extrapolated line.

The (intermediate) results of the pipe-line look like the following:

#### Step 1: Converted to grayscale
![alt text][image1]

#### Step 2: Blurred
![alt text][image2]

With the blurring I found the most usable results with a kernel size of 7.

#### Step 3: Canny edge detection
![alt text][image3]

For the canny edge detection, I used low threshold of 50 and a high threshold of 150. Lower than this resulted in many false positives.
With higher values, the lines were not properly detected. 

#### Step 4: Region of interest mask applied
![alt text][image4]

For the region of interest a polygon is applied from the lower left corner to a line almost centered in the middle of the image to the lower right hand corner:

```[(0, shape[0]), (shape[1] * 0.5 - 20, shape[0] * 0.6), (shape[1] * 0.5 + 20, shape[0] * 0.6), (shape[1], shape[0])]```

#### Step 5: Hough transform applied
![alt text][image5]


For the Hough transform the following parameters yielded the best results on the test set:

```
    rho = 1
    theta = np.pi / 180 * 1
    threshold = 15 
    min_line_len = 40 
    max_line_gap = 40
```

#### Step 6: Lines averaged and extrapolated
![alt text][image6]

The extrapolation and averaging step makes sure the final line follows the direction of the lane marks. 
Adding the lenght as weight to the averaging step makes sure that the line fits best in the near field (where the lines 
are generally longer).

#### Step 7: Final result overlayed on the original image
![alt text][image7]

#### Video results

##### White lines
[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/h-o_7kfa-Nk/0.jpg)](http://www.youtube.com/watch?v=h-o_7kfa-Nk)
[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/xe6jcJvGLiY/0.jpg)](http://www.youtube.com/watch?v=xe6jcJvGLiY)

##### Yellow left line
[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/lBhB6kurOqo/0.jpg)](http://www.youtube.com/watch?v=lBhB6kurOqo)
[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/JoyXVaQqbdE/0.jpg)](http://www.youtube.com/watch?v=JoyXVaQqbdE)


### 2. Potential shortcomings of the current pipeline

The current pipe-line has a number of shortcomings:

1. Faded lines are not well detected and can cause gaps. Only well defined lines will be detected correctly.
1. Curved lines are not supported with the Hough transform and can lead to strange results as this particular Hough transform only deals with straight lines.
1. Shadows can cause the line not to be detected. Shadows (or other sources of darkness in the video) can cause the line to not be detected well as it falls outside the parameters for the Canny transform.
1. When changing lanes, the lines will not be well detected. Due to the simple devision in a right and a left line marking, there is a moment in changing lanes where there will be more or less than two markings in the frame.
1. Random lines/cracks in the road surface may cause the lines to be slightly off. As the lines are only filtered in a basic way, random noise, within the area of interest, can make the lines not match up completely.
1. The resulting lines can be a bit jittery. As the videos are analyzed on a frame-to frame basis, the extrapolated lines can vary a bit, resulting in a, sometimes, unsteady image.


### 3. Possible improvements to the pipeline


1. To ensure that faded lines are detected correctly (to a certain point), the parameters of the blur and Canny transforms might still be adjusted better. Although making it more sensitive can also lead to more false positives.
1. To detected curved lines, a different form of / alternative to the Hough transform might be used that deals with curves specifically
1. Possibly, pre-processing of the video/images can help with shadows.
1. To make sure that changing lanes still displays the lines in the region of interest correctly, it might be better to separate the detected lines into an arbitrary amount of buckets instead of assuming a right and left lane at all times.
1. To ensure that random noice on the road surface does not affect the extrapolated lines negatively, better filtering might be added to make sure the extrapolated lines fall in the same general direction.
1. To improve on the jitter in the videos, the frame history could be taken into account to stabilise the line extrapolation 
