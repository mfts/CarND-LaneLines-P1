# **Finding Lane Lines on the Road** 

## Setup

### Installation

Runs Jupyter Notebook in a Docker container with `udacity/carnd-term1-starter-kit` image from [Udacity][docker installation].

```
cd ~/src/CarND-LaneLines-P1
docker run -it --rm -p 8888:8888 -v `pwd`:/src udacity/carnd-term1-starter-kit
```
Go to `localhost:8888`


## Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

**My pipeline**

My pipeline consists of 6 steps. First, I convert the image to grayscale, then I apply a Gaussian blur to the image. Step 3 applies the Canny Transform; I've chosen a `low_threshold=1` and `high_threshold=300`. Before drawing the Hough lines, I limited the area in a polygon shape where the lanes are. Finally applying the Hough lines took a little more iterations and fine tuning _(see my paramters in the code)_ to get good enough. Step 6 is just overlaying the original image with our Hough lines.

![alt text][image0]
Original image

![alt text][image1]
Hough Lines

A big part of generating Hough lines is **drawing the lines** themselves. Well, it's good enough if you already have a continuous straight line (like the far left/far right lane lines), but lane lines are typically dashed. In order for a self-driving car to know where it has to go at all times, it makes more sense to extrapolate the dashed lines to a straight continuous line as well.

The first thing that came to mind is **splitting the lane lines** into left and right by their slopes, respectively. If the slope is negative I assigned it to the right line and if the slope is positive I assigned it to the left line. I added a moving average over the slopes in order to include only the points that fell within a tolerance level of 0.1 divergence from the (left/right)SlopeAvg.

Over the arrays of 2D tuples I ran the OpenCV function `cv2.fitLine(points, distType, param, reps, aeps[, line])` in order to get vector of 4 elements `[vx, vy, x, y]`, respectively x-vector, y-vector, and a point on the line defined x and y coordinates. With this information I was able to calculate the slope and intercept of both lines.

I appended the four variables `(rightIntercept, rightSlope, leftIntercept, leftSlope)` to a [deque][deque], which is faster than a list in terms of memory and can be limited to a maximum length. I chose a `maxLen` of 5, which means the last 5 frames will be saved to the deque. I averaged the intercepts and slopes over the length of the previous frames.

Next, I set the start and end values for the y-axis. Basically limiting the frame from the bottom of the frame to around 2/3 into the frame vertically. By setting the y values I am able to calculate the start and end x values for both lines according to their intercept and slope.

Finally, I have all information to draw to an individual line for the left and right lane lines with the OpenCV function `cv2.line(img, pt1, pt2, color[, thickness[, lineType[, shift]]])`.

![alt text][image2]
Original image and Hough lines

**Applying the pipeline to videos**
Using MoviePy's function `fl_image` – a filter to modify a series of frames – takes my function `process_image(image)`, which is just the pipeline `lane_finder(image)` used for the image annotation described above. Please find annotated video files in in *test_videos_output*.

**Alternative extrapolation methods**
An alternative method for extrapolating the lines is using numpy's `polyfit()` function, which would deliver a similar result for a basic example but less sophisticated for a more complex frame.


### 2. Identify potential shortcomings with your current pipeline

The images and videos the pipeline has been tested on show mainly straight roads with clearly visible lane lines. Hence the pipeline and in particular the parameters in the Canny edge detection and Hough lines detection are biased. I ran the pipeline against the _challenge_ video, which shows a curved road with lots of noice near to the left lane. Without adjusting parameters in the Canny edge and Hough lines detections, the annotations spuriously change directions at one point in the output video.

Currently the polygon – limiting the area onto which Hough lines detection is applied – is a fixed shape, which assumes that there won't be a significant change in the road/image.

One obvious shortcoming is if there are no lane lines, i.e. if it's dark outside or a longer bridge/tunnel without dimming the images so that no lane lines can be detected.


### 3. Suggest possible improvements to your pipeline

One of the biggest improvements would be to continue tuning the Canny edge and Hough lines parameters. 

Instead of extrapolating a straight line, it may also be possible to extrapolate a polygon line based on the slopes of the previous lines/frames. That way annotating curves becomes more feasible and defensible against noise near the lanes.


[docker installation]: https://github.com/udacity/CarND-Term1-Starter-Kit/blob/master/doc/configure_via_docker.md
[deque]: https://docs.python.org/3/library/collections.html#collections.deque
[image0]: ./test_images/solidWhiteCurve.jpg "Original Image"
[image1]: ./test_images_output/report_hough_solidWhiteCurve.jpg "Hough Lines"
[image2]: ./test_images_output/annotated_solidWhiteCurve.jpg "Hough Lines on image"