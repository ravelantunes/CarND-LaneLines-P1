#**Finding Lane Lines on the Road**

This is my first project for the Udacity Self-Driving Nanodegree. This project was challenging and fun to work on, and forced me to a lot of catch-up with certain skills that I haven't had the chance to use much before like Numpy, Hough Transform, OpenCV, and Jupyter Notebook.

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

First, I applied different thresholds for different colors channels. With the exception of blue, I was able to apply very aggressive thresholds to the colors. Blue seemed to be abundant through the entire image, and not just in specific areas.

<!-- ![alt text][image1] -->
![color threshold][output_images/color_threshold.png]

Next was making the image in grayscale where the lines, regardless of color, would standout well, as well as making it easier to work on a single color dimension as opposed to 3.

![color threshold][output_images/grayscale.png]

Gaussian blur was applied to smooth the edges and bring up some weak points and lines.

![color threshold][output_images/gaussian_blur.png]

Canny edge detection was applied to find the contour of most lines. I found that my images after the color and grayscaling processing weren't very sensitive to the edge detection.

![color threshold][output_images/canny_edge.png]

Created a polygon mask to filter out all other areas of the image that are not relevant. I created dynamic values to support different image sizes, and also left some margins on bottom left and right to give some wiggle room for the when not perfectly centered in a lane.

![color threshold][output_images/region_of_interest.png]

I then identified lines through Hough Transform. I've kept the values low on threshold, minimum line size and maximum distance. Trying to keep those values high would filter out a lot of important information when working on yellow lanes, or shades on the road.

![color threshold][output_images/hough_lines.png]

Hough lines were used to create a single line for each side of the lane.
* Before lines are separated, I first find what's the smallest Y point (highest on the image) so I can use later to draw the image
* All lines are then separated based on the slope. Lines that had a slope between -0.4 and 0.4 were ignored as any slope on that direction of that car was likely noise.
* For each individual side, I calculated a weighted average of the slope, where the weight of the slope is the length of the line to the power of 5
* A list with all the points is created so a mean point can be identified. The mean point is used together with an imaginary point below the bottom of the image to calculate the slope of the line.
* Using the calculated slope, I draw a line from the imaginary bottom point to the top most point extracted before the list were separated.

![color threshold][output_images/final.png]


#### Challenge

I was able to get decent results very quickly while working with the main 2 videos. However, my initial approach had poor performance on the challenge video. The issues were:
- The patch of the road that is gray didn't provide enough contrast for the yellow lines to be detected.
- Some cars on the opposite side of the road would sometimes be identified as spots or lines.
- After the gray patch of the road, there were a few lines under the tree shade that didn't had bad contrast.

I had to make the following changes to generalize better for all videos:
- Decrease thresholds for colors and for Canny edges to get more lines under bad contrast situations.
- Modified how I calculate the region of interest so it used more dynamic values, as the challenge video had different dimension that the other videos.
- Create a more robust slope average calculation of the hough lines to compensate for the additional noise of the low thresholds.

###2. Identify potential shortcomings with your current pipeline

A few shortcomings that I believe would be an issue with my approach is the fact I relied heavily on the color thresholds. That might have worked well for the landscape of the images, which were kind of similar between all the videos, but would not translate well with different weather or even sunlight conditions.


###3. Suggest possible improvements to your pipeline

- Right now, each frame is ignorant of the state of the previous frames. An algorithm like Kalman filter or even something more simple as a moving average would help smoothing out the line changes frame-by-frame and the impact of noise.
- Lines towards the bottom of the image are much easier to recognize, and could make sure of stricter threshold for things like colors, Canny edges, and Hough lines, while lines towards the middle required smaller threshold to identify lanes, and as consequence ends up capturing a lot of noise. That could be overcome by applying different set of thresholds to different parts of the image, or through a gradient.
