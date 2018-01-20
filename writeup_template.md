# **Finding Lane Lines on the Road** 

## Writeup

### This is a writeup for the project to find lane lines on the road and highlight them.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on the work in a written report


[//]: # (Image References)

---

### Reflection

### 1. Pipeline description.

The pipeline I have developed consists of 6 steps.

* First I read the image and converted it to grayscale.

![gray](./md%20resources/solidWhiteRight_gray.jpg)

* Gaussian blur is applied to grayscaled image to filter out noise.

* The edges are detected using canny edge detection algorithm. The result is an image with only edges.

![canny](./md%20resources/solidWhiteRight_canny.jpg)

* Next I created a region of interest and applied it over the image shown above. In the output I got an image with only the necessary information i.e. lane lines.

![masked](./md%20resources/solidWhiteRight_masked.jpg)

* In order to find out the all the x,y coordinates that belong to the lane lines I took a hough transform of this image. The hough transform provided the coordinates of start & end point of all the lines that are detected in the image. 
Lines corresponding to left and right lanes are identified and drawn (more info on drawing lines below). 
The output is as shown here.

![hough](./md%20resources/solidWhiteRight_hough.jpg)

* This image is overlayed on top of original image to highlight the lanes which are detected. :smiley: The result is :

![final](./md%20resources/solidWhiteRight.jpg)

---

#### Modifications to draw_lines() function
This is how I modified the draw_lines() function in order to draw a single line on the left and right lanes.

First I seperated the points into 2 groups, one for points on left lane and other for points on right lane.
```python
# seperating the points belonging to left and right lane
    for line in lines:
        for x1,y1,x2,y2 in line:
            if(x1 < 480):
                left_x.append(x1)
                left_x.append(x2)
                left_y.append(y1)
                left_y.append(y2)
            else:
                right_x.append(x1)
                right_x.append(x2)
                right_y.append(y1)
                right_y.append(y2)
```
After seperating the points I converted the list to numpy array.
```python
    np_left_x = np.array(left_x)
    np_left_y = np.array(left_y)
```
Next, using `polyfit()`, I found the cofficients (**A** and **B**) of a first order polynomial equation that would fit all the points for each lane.
```python
    left_lane = np.polyfit(np_left_x,np_left_y,1)
```
Using these cofficients I was able to model a single line for both left and right lanes as  
**_y_ = A _x_ + B**

In this equation I put the y coordinates of my Region of interest to obtain x values.
```python
    x1 = int((y1 - left_lane[1]) / left_lane[0])
```
I passed these values to `cv2.line()` to draw a single lane line.
```python
    cv2.line(img, (x1, y1), (x2, y2), color, thickness)
```
The output images are saved in [test_images_output](https://github.com/nikhil-sinnarkar/CarND-LaneLines-P1-master/tree/master/test_images_output) folder and the output videos are saved in [test_videos_output](https://github.com/nikhil-sinnarkar/CarND-LaneLines-P1-master/tree/master/test_videos_output) folder.

---
### 2. Potential shortcomings with the current pipeline


The threshold values for grayscale, canny edge detection and hough transform are set manually so the pipeline may not be able to identify lanes under different lighting conditions. For example in cases when there is shadow over the road or abrupt changes in tarmac color.

The boundaries of Region of interest are also set manually. These may not work for 
  1. Images having different resolution (width, height) and  
  2. Changes in camera orientation. For eg. if camera is pointed slightly upwards.
---

### 3. Possible improvements to the pipeline

Currently the pipeline detects straight lines. The pipeline can be improved to also detect curved lane lines.

Improvements can also be done to smooth out the lane detection. This can be achieved by averaging the values detected in previous frames of the video.

Some logic can be incorporated into the pipeline to dynamically change the threshold values and to detect and set the region of interest.
