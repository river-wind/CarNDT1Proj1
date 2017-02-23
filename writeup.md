#**Finding Lane Lines on the Road** 
##Writeup for Chris Lawrence's first project for CarND Term1
###

---
**Finding Lane Lines on the Road**

The goals / steps of this project were:
* Make a pipeline that finds lane lines on the road
* Reflect on the work in a written report

[//]: # (Image References)

[image1]: ./examples/grayscale.png "Grayscale"
[image2]: ./examples/candidates.png "Candidate lines"
[image3]: ./examples/twoLines.png "One line per side"
[image4]: ./examples/finalImage.png "Final image"
[image5]: ./examples/problemLine.png "Line Estimation Error"

---

### Reflection

###1. The pipeline I have created takes 6 main steps, most of which were reviewed earlier in this lesson to one degree or another.
####a.      First, the image is simplified down to greyscale to reduce the complexity of the problem.
![alt text][image1]
####b.	    Next, the image is blurred with the gaussian_blur function to reduce image noise, using an odd-numbered kernel.
####c.	    The resulting image is then used as the source for the Canny edge detection algorithm.
![alt text][image2]
####d.      The edge-detected image is then reduced using a vertex array and the polyfill function to build a collection of candidate lane lines. 
####e.		A blacked-out copy of the image is then created (to preserve image dimensions), and the candidate lines are applied to that copy using the Hough line process.
![alt text][image3]
####f.		Finally, the image with the lines applied is merged with the original color image file for export
![alt text][image4]

This pipeline relied on a number of support functions which were provided to us.  Those functions included code previously scattered across the example code blocks used in different quizzes throughout the lesson.  From the region_of_interest(), hough_lines(), to the weighted_img() function, the main pipeline code was greatly streamlined by the use of these functions.  

One of the functions, draw_lines(), did a fair job of drawing the candidate line segments to the blacked out image out of the box.  What this function did not do, however, was a roadblock to progressing from a fair job to approaching the example output video provided.
####a. The function did not extend the edge-detection based lines to the bottom of the image.
####b. The function drew multiple line segments per actual lane line, rather than the desired single solid line as demonstrated in the example output movie file.

To resolve these limitations, I first found the line slopes and y intercepts.  Rather than bothering to extend the existing lines to these top and bottom positions, I next reduced the total number of lines to just two - one left and one right line.

To reduce the multiple lines per side to a single line per side, I built 4 lists of extrapolated x values.  A list for the right line's x value at the top of the region of interest (in this case 325 pixels), one for the x values at the bottom of the image (maximum y), as well as the left line's top x and bottom x points.  Once I had these four lists, I could take the mean value of each, leaving me with four points: the top and bottom points for the new left line, and the top and bottom for the right line.  Those can then be used in conjunction with the cv2.line() function to draw the new lane lines onto the blacked out image from step e in the pipeline.


###2. Potential shortcomings with the current pipeline

This pipeline has a number of limitations, including falling victim to low-contrast conditions such as would occur during a rain or snowstorm.  It also relies on a straight line for lane predictions; strongly curving roads would not produce a very accurate prediction.  Steep uphills will result in the road taking up a larger portion of the image than normal, which would cause the region_of_interest() function to improperly ignore important information through its use of a hard-coded upper threshold.

There was an outright error which occurred in the yellow example movie.  A short white line crossing the yellow perpendicularly caused the prediction to go completely wrong, rotating the line 90 degrees from the correct direction.
![alt text][image5]

I encountered another likely limitation of this method came up during my first experience in a car offering semi-autonomous driving this week.  In a construction zone, the dashed lane lines were replaced with staggered series of reflectors.  These were positioned such that they mimicked dashed white lines, however the car itself could not recognize them at all, and disabled its autonomous driving function completely.  I could imagine that the multiple gaps between reflectors could make it difficult for the Hough Line process to accurately identify them as line segments, despite its success in managing the gaps between the dashed lines.  Reducing the minimum line sgement threshold to a small enough value that would allow the reflectors to be recognized could cause a large increase in false positives being triggered by any small white object without sufficient management.

###3. Possible improvements to the pipeline

There are a number of things which could improve the process as it currently stands.  Rather than creating a straight line per side, using a Bézier curve to represent the lines would more accurately reflect reality.  The two lanes could be better represented by effectively parallel curves, transformed to account for the vanishing perspective of the road.

In addition, by reducing the image to greyscale, the processing is simplified, but data is lost.  Running the process against the three color channels individually and averaging the three resulting predictions instead could increase the accuracy of the final result.

Lastly, to resolve the erroneous prediction which occurred in the yellow line example, the line predictions could either be artificially limited to a certain angle range, or the predictions could be averaged across frames to reduce the chance for wide prediction swings occurring rapidly due to what amounts to noise across frames.  The former solution would resolve this issue but would cause actual crossing lane lines or cross walks to be ignored.  Averaging across frames could resolve individual frame errors as well as reduce the impact of a few frames of strange noise, but a cross-walk like situation would still not be handled.  In the end, a combination of processes would be needed to eliminate this error while not ignoring important perpendicular lines.
