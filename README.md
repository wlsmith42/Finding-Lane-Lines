# **Finding Lane Lines on the Road** 
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road in images
* Apply the lane finding pipeline to videos
* Improve the draw_lines() function to output a single solid line over each lane 


[//]: # (Image References)

[image1]: ./test_images/solidYellowLeft.jpg "image"
[image2]: ./test_images_output/solidYellowLeftHLS.jpg "hls"
[image3]: ./test_images_output/solidYellowLeftColorSelect.jpg "colorSelect"
[image4]: ./test_images_output/solidYellowLeftEdges.jpg "edges"
[image5]: ./test_images_output/solidYellowLeftMaskedEdges.jpg "maskedEdges"
[image6]: ./test_images_output/solidYellowLeftResult.jpg "result"


---

## Reflection

### Pipeline Description

My pipeline consisted of the following steps:
* Convert image to HLS
* Run color selection algorithm to isolate yellow and white
* Convert image to grayscale
* Run Gaussian smoothing
* Find edges using Canny Edge Detection
* Apply region of interest mask
* Draw lines from points with Hough transform
* Overlay lines on original image

This is one of the starting images used to test my pipeline. There were no shadows or different road surfaces to deal with, so line detection is fairly straightforward. 
![alt text][image1]

The first step was to convert the image from RGB color space to HLS color space. For this particular image, this process won't make too much of difference but for less favorable conditions it is very useful. When the road is shaded or white lines are on a concrete road, converting to HLS often results in a much higher contrast between the lines and the road which improves the efficiency of Canny Edge Detection later on.
![alt text][image2]

Now that the lines are clearly visible, I remove the extra information in the image. To do this I run a simple color selector for white and then yellow and combine to the two color masks. This results in an image that only has white and yellow pixels.
![alt text][image3]

Next I convert the image to grayscale. One interesting outcome of this step is that the image in HLS color space does not have the same grayscale as an image in RGB color space. Overall this color space issue did not affect the final result so I left this step in the pipeline although it may not have been completely necessary. 

After the image is converted to grayscale, I apply Gaussian smoothing to help smooth out color gradients to improve the accuracy of the Canny Edge Detection.

At last the image is ready for Canny Edge Detection! I used a low threshold of 50 and a high threshold of 150 and that seemed to do a good job. 
![alt text][image4]

Once again, to help remove extra information, I apply a mask to the image so that the only pixels remaining reflect the lines on the road. This filters out all of the white and yellow pixels elsewhere in the image that would otherwise alter the line averaging in the next step.
![alt text][image5]

The last step is to convert the points generated in Canny Edge Detection to lines using a Hough Transform. The result of this is a solid line surrounding the lane lines. From here, the lines need to be averaged to create a single left line and single right line. 

The line averaging is computed within the draw_lines() function. To accomplish this, I simply group each line by its slope. A negative slope means it is the left lane line and a positive slope means it is the right lane line. Once grouped, the lines are averaged using the dot product and weighted based on the cumulative length of each line group - longer lines have more weight in the average.

Finally, the lines are overlaid onto the original image giving us a full view of what the pipeline detected.
![alt text][image6]


### Shortcomings of the Pipeline

One potential shortcoming would be how the line jumps around when detecting a dashed line. This is simply due to how the dashed line comes into the pipelines field of view and the addition of a new line segment causes the draw_lines() function to adjust the lane line's slope, causing a slight jerking motion.

Another shortcoming could be how white lines are detected on lighter roadways such as concrete. In the challenge video (test_videos_output/challenge.mp4), as the car transitions from the concrete bridge to the asphalt road, the left lane line briefly disappears. This is due to the threshold that is applied in the color selection algorithm. I did a lot of experimenting with this value, but lowering the white threshold anymore caused too much noise that ultimately reduced the accuracy of the lines. 


### Improvements to the Pipeline

A possible improvement would be to fix the line jerking by replacing the single lane line with a single lane curve. This should prevent the detection of new dashed lines from causing the lines slope to vary. This would also allow a much smoother line to be drawn throughout curvy roads.

Another potential improvement could be to reducing line blinking by combining HLS color space with RGB color space. Since some lighting conditions prove difficult in certain color spaces, the addition of more color spaces could improve the pipelines ability to see the lines in all lighting conditions.
