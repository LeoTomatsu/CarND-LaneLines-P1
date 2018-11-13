# **Finding Lane Lines on the Road** 

### by Leo Tomatsu
### contact: leotomatsu@gmail.com

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

#### Pipeline description
Please see the function "pipeline" in "P1.ipynb" for more information.

My pipeline consisted of 5 steps.

##### 1. Read in and grayscale the image.
![alt_text][test_images/output/solidYellowLeft/Grayscale_Image_solidYelowLeft.jpg]

This returns an image with only gray color channel. I returned the grayscale image.
##### 2. Apply Gaussian smoothing to image.
![alt_text][test_images/output/solidYellowLeft/Gaussian_Blur_Image_solidYelowLeft.jpg]

I supress noise and spurious gradients by applying Gaussian smoothing. In this case, I used 3 as my `kernel_size` for Gaussian smoothing. I return the image with Gaussian smoothing appliedd.
##### 3. Apply Canny edge detection to image.
![alt_text][test_images/output/solidYellowLeft/Canny_Edge_Image_solidYellowLeft.jpg]

The Canny algorithm will first detect strong edge pixels above the parameter high_threshold, and reject pixels below the low_threshold. Next, pixels with values between the low_threshold and high_threshold will be included if they are strong edges. After some testing, I defined the high_threshold as 60 and low_threshold as 180 to be able to draw lane lines that were yellow. The output edges is a binary image with white pixels tracing out the detected edges and black background. I return the edges as an image.
##### 4. Apply hough transform to masked four sided polygon image.
![alt_text][test_images/output/solidYellowLeft/Hough_Lines_Image_solidYellowLeft.jpg]

First I defined my four sided polygon to mask the image. I determined my verticies as follows: `(0, xsize), (xisze / 2 - 50, 3*ysize / 5), (xsize / 2 + 50, 3*ysize / 5), (xsize, ysize)`. The first and last points were determined by the bottom corners of the image. The second and third points were set to be able to identify the left and right lane separately for a given image. I took the points that I orignally found that were hard coded, and added the formula to fit the ratio that I found between the hard coded value and the image size. Then, the region_of_interest function only kept the region of the image defined by the polygon fromed from the vertices. It also turned the rest of the image to be black.
Second, I set the parameters to call the hough_lines function. The parameters for this would be as follows: `rho = 1`, `theta = np.pi/180`, `threshold = 1`, `min_line_length = 10`, and `max_line_gap = 5`. These parameters were valued after testing which values drew the most fit line. Next, I call the draw_line function. I describe the modifications I made in draw lines below, please reference in title "draw_lines_modification". After the lines are drawn, I return the hough lines image.
##### 5. Draw the lines on the edge image.
![alt_text][test_images/output/solidYellowLeft/Hough_Transform_Image_solidYellowLeft.jpg]

Finally, I draw the lines on the edge image using the `weighted_img` function. This function combines the original image with the image with the masked hough lines. I returned the hough transformed image.

#### draw_lines modification
Originally, the `draw_lines` function would draw every line in its parameters. This caused
the image to appear to have lines that would not fit the lane line in the original
image.

In order to improve the fit of the line, I modified the `draw_lines` function by separating the line segments by their slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left line vs the right line. I achieved this by first filtering out any zero division error cases, then defining negative slopes and x-axis variables less than half of the image x-axis shape to be in the left line. Thus, positive slopes and x-axis variables larger than half the image x-axis shape to be in the right line. After the binary separation of left and right, respectively I find the best-fitting line for given points using the function `best_fitting_line` in "P1.ipynb". I added a helper function that averaged the position of each of the lines and returned points that extrapolate to the top and bottom of the lane. In more detail, I created two lists for x-axis and y-axis points. For each line, in order I appended each points to the list that made a line. Next, I found the slope and intercept with the x-axis and y-axis lists by utilizing the polyfit function in numpy. Polyfit returns the coefficients for a polynomial p(x) of degree of 1 (in our case) that is a best fit line for the data in the y. After receiving the best fit points for y, I derive the x coordinate using the equation of the lines and derived values. This was determined by the best fit x points by subtracting the best fit y points with the intercept, and dividing with the slope to get x. Finally, I return the best fit lines for x and y for right and left.

#### Side note
For images, I created folders for each image to store images from all five steps in my pipeline.
For videos, I did not modify the file structure or filename. Please see 

### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when the line would not properly fit line segments. For example, please see the hough transform image for solidWhiteRight below.
![alt_text][test_images/output/solidWhiteRight/Hough_Transform_Image_solidWhiteRight.jpg]
The left line appears to be drawn too narrow comperative to the actual left line. This may have occurred due to the lack of lines created during my pipeline process. Since the solid-lane line appeared to have been drawn accurately, have a secondary set of parameters for lines that could be identified as segmented lines may have improved the best-fit line for dotted-lane-lines.

Additional shortcomings would be how I managed the yellow lane lines. When I viewed the masked edges iamge for lanes that were yellow, there were sigificantly less lane lines available for extrapolation compared to white lines. I thought that if I could identify other lane colors that were not yellow, be adjust to that color or filter, I would be able to improve the collection of lane lines.

Another shortcoming could be that my modified draw_line function was unable to best fit lane lines that were curved. This phenomenon was found when I tried the optional challenge. Since my best fit line was created for linear functions, it could not properly trace the curve. I began creating a new pipeline function called "pipeline_curve" to be able to adjust to this new input. I thought that if I found the coefficients for a parabola using the given lane lines I would be able to estimate the x and y points. I did not have time to improve the parameters and other configuration to properly draw the best fit curve. I would need to update parameters and look at why the curves are not directly drawn over the lanes.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to separate parameters for segmented lane lines and solid-lane lines in the function `draw_lines`. I believe if I could identify asegmented lane line from a solid-lane-line, possibly from the number of lines, I would be able to improve the best-fit line for segmented lane lines. As stated above in my short-comings, the segmented lane line would produce draw lines that would be properly line the lane line.

To increase the lane lines from yellow lane lines, I believe I could try other color filteration to be able to identify yellow and other potential lane colors other than white.

Another potential improvement could be to adjust my parameters and filter the lane lines to be able to draw the parabola curves. As stated above, I unfortunately ran out of time to continue editing parameters and looking for a better method to estimate the curve line. Please see the below list of TODOs that I thought I have to improve.
+ Improve x-axis estimates by finding minimum and maximum x-axis points for each lane line.
+ Improve polynomial function from parabola.
Please see the function "pipeline_curve" in "P1.ipynb" for more information.

### Citation
+ Curve Fitting and Plotting in Python: Two Simple Examples, sdtidbits, http://sdtidbits.blogspot.com/2009/03/curve-fitting-and-plotting-in-python.html
+ numpy.polyfit, SciPy.org, https://docs.scipy.org/doc/numpy-1.15.0/reference/generated/numpy.polyfit.html
+ Non-Linear Least-Squares Minimization and Curve-Fitting for Python, LMFIT, https://lmfit.github.io/lmfit-py/model.html