# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on the work in a written report


[//]: # (Image References)

[grayscale_img]: ./test_images_output/intermediate/grayscale.jpg "Grayscale"
[edges_img]: ./test_images_output/intermediate/edges.jpg "Edges"
[masked_edges_img]: ./test_images_output/intermediate/masked_edges.jpg "Masked Edges"
[lines_img]: ./test_images_output/intermediate/lines_img.jpg "Lane Lines"
[final_img]: ./test_images_output/solidWhiteCurve.jpg "Final"

---

## Reflection

### 1. The image process pipeline

The pipeline consisted of the following steps:

* Convert the image to grayscale.
![alt text][grayscale_img]



* Using Guassian smoothing/blurring, suppress and reduce noise and spurious gradients by averaging.
* Detect edges using canny detection method from cv2.
![alt_text][edges_img]



* Create a masking filter for the region of interest where we want to detect the lane lines.  
![alt_text][masked_edges_img]



* Using hough lines detection API from cv2, identify and draw the lane lines. 
The task of improving the draw_lines() function ended up consuming the bulk of the time spent on the project. 
While using the cv2 hough line API to identify line segments was as simple as making a single function call, 
processing the resultant list of detected line segments required a bit more work. In order to draw over the 
lane lines, I first found the slope of each line segment from the result set of detected line segments. Based
on the sign direction (Note that in coordinate system where y-axis values increase as we go from top to bottom,
a negative slope indicates a line that go from the lower-left up to the upper-right) of the line segement slopes,
the list of line segments were then separated into two lists--one for the left and one for the right lane lines.
Continuing, I processed each list of line segements by finding the midpoint of each line segment. Then using numpy's
ployfit() function a best line was fit to the midpoints (average line segments). The result of this was very
good when lines were drawn on the test images. However, when drawn on successive frame images of the test videos the
resulting output videos looked very jittery with the lane lines moving all over, sometimes with lines that seem to
cross over to the other side and with some even being almost horizontal. This poor perfomance on the video clips
required some review and rework of my first extrapolation strategy.

  Iteration 2: Using the same segregated (left-right) lists, instead of finding the midpoints of the line segments,
  this time around, I used the numpy's polyfit() function to determine the line (i.e., get the slope and intercept) 
  based the two endpoints of each line segment. From the list of slopes and intercepts, the average slope and average
  intercept was found. Then using the line determined by the average slope and average intercept, I drew the new
  lines over the image lane lines. This time, the videos were looking somewhat better, with the lines mostly staying 
  on top of the actual lane lines. But there were still occasional frames where the lines would just jump wildly
  unexpectedly.
  
  Iteration 3: The way some of the lines seemed to be pulled way off to the opposite side of the screen made me think
  that perhaps some of the detected line segments' slopes differed greatly in magnitude than the rest of the line
  segments, i.e., there must be some outlier line segments that were pulling the averages in an undesireable way. So,
  I implemented a simple function acceptable() to filter out line segements from the list with slope magnitudes that
  fall within a range of 0.55 and 1.0. The rest of the extrapolation strategy was left the same as iteration two. In
  filtering out the "outlier" line segments a lot of noise was removed and thus, resulted in the much more stable
  lane lines drawn in the final output videos. This latest iteration even worked out pretty well for the challenge
  video clip. Updating the region of interest to have vertices that were relative percentages of the input image 
  shape instead of the original hardcoded number values also helped improve the accuracy on the challenge video.
  ![alt_text][lines_img]


* Superimpose the detected lane lines onto the original road image
![alt_text][final_img]






### 2. Potential shortcomings with the current pipeline


A potential shortcoming with the current pipeline is the fixed range used for filtering for acceptable slope values.
The hardcoded range endpoint slope values may be throwing away relevant line segments or conversely, including invalid
line segments.

Another potential shortcoming with the pipeline may be the region of interest. Currently, using relative percentages of
the input image shape assumes that the lane line is within a certain area of the image. What happens if the image is
captured while the camera angle moves, causing the frame around the lane lines to change beyond the specified region?


### 3. Possible improvements to the pipeline

A possible improvement to the pipeline may be to refactor the acceptable() filter function by getting rid of the fixed
range for the slope value. Instead, the function may imploy a method for outlier detection by checking a target
line segement's slope against all the slopes of the rest of the line segments within the list of all potential line
segments in the list.
