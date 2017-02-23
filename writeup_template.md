#**Finding Lane Lines on the Road** 

##Writeup Template

###You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 8 steps:

1. Convert the images to grayscale
2. Run a Gaussian blur on the grayscale image to smooth out the noise in the
   image. 
3. Run canny edge detection to isolate the edges in the image. 
4. Apply a mask to the canny edge image to focus on the area directly in front
   of the car. This obviously assumes the camera angle is fixed.
5. After the mask is applied, run a hough transform to determine the major lines
   in the area of interest. 
6. The lines detected by the hough transform are split into those falling on
   the left and right sides by taking both the line segments slopes and
   x-coordinates into consideration. 
7. Run RANSAC on the line segments (per side) to predict the linear
   model, while ignoring the outliers. 
8. Use the RANSAC model to predict a line segment, which allows one to solve for
   the slope (`m`) and `b` for the line equation `y = mx + b`. From that the two
   end points for a `y` at the bottom and a `y` near the middle of the image can
   be calculated.

I tuned the hough parameters to allow *more* line segments rather than less to
give RANSAC more data points. Additionally, I 'remembered' the hough lines from
the previous run (in the case of video) to smooth out the jitter in the detected
lines. 

###2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when changing lanes. The
area of interest mask is too narrow to properly track the lanes during a lane
change.

Another shortcoming is that the predicted lines area straight, and not curves.
This would break down when turning aggressively. 

This is also tested on a small sample set, so the parameters are tuned to the
data set and would likely need to be retuned given different landscapes, road
conditions, and lighting conditions.

The current pipeline also does not fair too well on the challenge problem.  The
camera seems to have a wider field of view, and as such the actual region of
interest is more narrow. As a result the hough transform detects additional
line segments that pass the slope and x-coordinate checks, and are sometimes so
numerous that they 'trick' the RANSAC algorithm. 

###3. Suggest possible improvements to your pipeline

To improve this it would likely be better to keep a longer 'memory' of past line
segments, instead of just the very last ones. These could then be weighted in
importance, the newest line segments being weighted higher than the older.

Additionally, we could use only the line segments that were considered as
inliers by RANSAC. We could also possibly use RANSAC to interpolate 'fake' data
points along the line to further reinforce the line that was previously being
tracked to prevent radical jumps in the line position.

Finally, I'm sure there is a fair amount still to be achieved by tuning the
various parameters already implemented to get better line segments to choose
from overall.
