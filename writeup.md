# **Finding Lane Lines on the Road** 

## Writeup for Project 1


---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps: 
 1) Convert the images to grayscale
 2) Apply a gaussian blur to reduce noise
 3) Use Canny edge detection to find edges using the intensity gradient
 4) Apply a region of interest mask to filter lines that aren't on the road
 5) Use Hough transform and find common intersections to identify line segments on the road
 6) Sort lines into left lanes, right lanes, and ignored lines by applying a slope filter
 7) Average slopes and offsets of sorted line segments to determine the final lane line

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by adding a slope filter. First, I determined the slope and offset of each line segment. If the slope were positive and above a threshold, it would contribute to the sum for the right line. If the slope were negative, it would contribute to the sum for the left line. I then divided each slope sum by the count of lines in each category to get an average slope. I followed the same method for finding the average offset. Once I had the average slope and offset, I drew a line with those values from the bottom of the picture to the horizon.

My reason for applying a slope filter rather than just sorting based on positive and negative slopes was that certain lane markings were perpendicular to the actual lane line. Those lines had the same features as the actual lane lines (white paint on asphalt) and therefore were easiest to filter based on slope rather than some other feature. Presumably, the lanes should be roughly in the direction of travel and any lanes that are perpendicular to the travel of the car wouldn't be that useful here anyway.


### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming of this pipline would be sensitivity to changes in the contrast of the input images, for instance at night or as the road surface changes like in the challenge video. The Canny edge detection is currently tuned for the intensity gradient distribution that was provided in the test images and videos and would not likely be good at finding edges at all other contrast conditions.

A related problem is that features with similar contrast in the region of interest will pass through the Canny edge detection and contribute to the slope and offset adjustment of the final lane line. A good example of this is how the shadows from other cars and lane barriers impact the quality of lane finding in the challenge video. Features like that outside the lane could be fixed by tightening the mask, but there can also be features like that within the lane that should be considered. One colleague noted to me that his Subaru with lane keeping assist will sometimes follow the tar lines instead of the paint markings, so I suspect that to be related to this issue.

Finally, my method of finding a best fit line is somewhat weak. An average will be thrown off by noise and is not the most robust method for finding a best fit. It was simply the easiest thing for me to implement based on my limited experience in Python.

### 3. Suggest possible improvements to your pipeline

To address the concern about contrast variation, a potential improvement would be to somehow normalize the contrast. My initial thought was to apply a local normalization in case the contrast range varies over the image as it does in the challenge video, but I suspect that there are better ways to manipulate the contrast. A quick search suggests that maybe histogram equalization could help. I intend to browse OpenCV and research some more on image processing to learn about the recommended techniques and then try a few to improve my performance on the challenge video.

To improve the noisiness from lane-like features, I could tighten the region of interest and also mask the interior of the lane. In an actual self-driving car, I would be hesitant to do that though. I think that the positional tolerance of the camera with respect to the lane markings will be quite large and that a wide region of interest is therefore required. At least during a lane change, an interior mask would block the actual lines entirely. So, this issue should be addressed through some less agressive filter than a mask. Perhaps a better way would be to apply a color filter before the conversion to grayscale so that white and yellow lines will have higher contrast than gray ones.

The susceptibility of the slope averaging method to noise would be mostly fixed by using a proper best-fit line method like polyfit, as suggested by someone in the Slack channel. There would likely still be some sensitivity to noise in that case, but it could be further improved by filtering outliers prior to the line fit.
