# **Vehicle Detection**
---

**Vehicle Detection Project 5**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: output_images/car_notcar.jpg
[image2]: output_images/car_notcar_hogs.jpg
[image3]: output_images/scaled_windows.jpg
[image4]: output_images/hot_windows.jpg
[image5]: output_images/heatmaps.jpg
[image6]: output_images/labeled_heatmaps.jpg
[video1]: project_video_output.mp4

---
## Rubric Points
#### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


#### README File

###### 1. This is it.

#### Histogram of Oriented Gradients (HOG)

###### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the A3 code cell of the IPython notebook.  The code is essentially the same as the code in the lesson, with a few slight modifications.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

###### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of HOG parameters, but finally settled on one with 9 orientations, 2x2 cells_per_block and 8 pixels_per_cell.  I used all 3 color channels for the HOG (in the YCrCb color space), as it seemed to perform the best.  A spatial size of 16x16 pixels seemed to work well without too much loss.

###### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the following features:  a spatial binning of the colors, followed by a histogram of the colors.  I determined that the YCrCb color map seemed to give the best detections.  Finally, I chose a HOG with 9 orientations, 2x2 cells_per_block and 8 pixels_per_cell.  After normalizing and concatenating into one feature vector each for both features and labels, I trained a LinearSVM and achieved almost a 99% accuracy rate.  I also tried to optimize the SVM with a GridSearch, and determined that a C of 10 would perform best.  This code is mostly in cells A3 and A4 of the notebook.

### Sliding Window Search

###### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

Code cell A5 contains the procedures for the sliding window search.  Again, much of the code is from the lessons with a few modifications.  I searched windows in 4 scales, and wrote a small function in which to easily change the scales so that I could try different ones.  Scales of 1.0, 1.25, 1.5 and 2.0 seemed to do ok, with the smaller scales searching only the uppermost area of the region of interest, while the largest searched the entire region of interest.

![alt text][image3]

###### 2. Examples of test images:

Here are examples on the test images.  You can see several false positive in the images.

![alt text][image4]
---

### Video Implementation

###### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
[Link to video](video1)


###### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

##### Here are six frames and their corresponding heatmaps:

![alt text][image5]


###### Here the resulting bounding boxes are drawn onto the frames:

![alt text][image6]

I used some additional functions to filter out false positives and provide smoother tracking of the vehicles.  Code cell A8 and A10 contain these functions.  I saved a list of good detections, possible detections and a draw_box list.  I saved detections as boxes, and then tested these boxes for size and distance of the center from any previous boxes.  I introduced a margin-of-error for each, and threw out any detections that didn't match (size +/- 50% and distance 15 pixels from previous detections).  This eliminated quite a few false positives.  Good matches were saved in a list, while possible detections were saved as well.  If a new detection matched a possible detection from the previous frame, it was assumed this would be a good match.  The draw_box list was updated every 6 frames to smooth out the detections.

---

### Discussion

###### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are still some issues with this implementation.  The classifier still has quite a few false positives and still misses some cars (especially the white car at the smaller scale).  I added some data to the noncar image set, but it seemed not to make much of a difference.  I think additional data of both car and noncar might help in making a more robust classifier.  The filter system is not fool-proof either.  A particularly fast car might be filtered out because of the margin-of-error on distance differences, and the check for the size of the detection rectangle could probably be improved as well.  The implementation takes about 2 hrs to process a 50s video, which is nowhere near real time.  This would have to be speeded up a great deal before use in any actual system.
