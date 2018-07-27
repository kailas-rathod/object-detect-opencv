# object-detect-opencv
---

<p align="center"> 
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/OpenCV_Logo_with_text_svg_version.svg/1200px-OpenCV_Logo_with_text_svg_version.svg.png" width=250 height=300>
</p>

***
---

# So Let's get started

---

## 1. cap = cv2.VideoCapture(0)


**Capture Video from Camera**

Often, we have to capture live stream with camera. OpenCV provides a very simple interface to this. Let's capture a video from the camera (I am using the in-built webcam of my laptop), convert it into grayscale video and display it. Just a simple task to get started.

To capture a video, you need to create a VideoCapture object. Its argument can be either the device index or the name of a video file. Device index is just the number to specify which camera. Normally one camera will be connected (as in my case). So I simply pass 0 (or -1). You can select the second camera by passing 1 and so on. After that, you can capture frame-by-frame. But at the end, don't forget to release the capture.

Sometimes, cap may not have initialized the capture. In that case, this code shows error. You can check whether it is initialized or not by the method **cap.isOpened()**. _If it is True, OK. **Otherwise open it using cap.open()**_.

**When everything done, release the capture**
 
 ###                        cap.release()
 ---
 
 
 ## 2. center_points = deque()
 
 It's used to store all the points that are traced path generated by the moving of the object in the front of camera. _Basically, the blue lines on the screen_
 
 ---
 
 ## 3. lowergreen = np.array([50,100,50])
##     uppergreen = np.array([90, 255, 255])

Defining the range of the specific color of the object so we can detect it. We will see how to use this later.

---

## 4. flip

*Flips a 2D array around vertical, horizontal, or both axes.*

* C++   : void flip(InputArray src, OutputArray dst, int flipCode)

* Python: cv2.flip(src, flipCode[, dst]) → dst

* C     : void cvFlip(const CvArr* src, CvArr* dst=NULL, int flip_mode=0 )

* Python: cv.Flip(src, dst=None, flipMode=0) → None

### Parameters:	
1. src – input array.

2. dst – output array of the same size and type as src.

3. flipCode – a flag to specify how to flip the array; 0 means flipping around the x-axis and positive value (for example, 1) means flipping around y-axis. Negative value (for example, -1) means flipping around both axes (see the discussion below for the formulas).

The function *flip* flips the array in one of three different ways (row and column indices are 0-based):

<p align="center"> 
<img src="https://docs.opencv.org/2.4/_images/math/c81497d41245321cc4906cde79d950d95f08ae8a.png">
</p>

The example scenarios of using the function are the following:

* **Vertical flipping of the image (flipCode == 0)** to switch between top-left and bottom-left image origin. This is a typical operation in video processing on Microsoft Windows* OS.

* **Horizontal flipping** of the image with the subsequent horizontal shift and absolute difference calculation to check for a vertical-axis symmetry (flipCode > 0).

* **Simultaneous horizontal and vertical flipping of the image** with the subsequent shift and absolute difference calculation to check for a central symmetry (flipCode < 0).

* **Reversing the order** of point arrays (flipCode > 0 or flipCode == 0).
 
 ### So in Short Flipping is done to mirror the camera output so you can easily work with it.
 
 ---
 
 ## 5. frame2 = cv2.GaussianBlur(frame, (5, 5), 0)
> .
* Image blurring is achieved by convolving the image with a low-pass filter kernel. It is useful for removing noises. It actually removes high frequency content (eg: noise, edges) from the image. So edges are blurred a little bit in this operation.
> .
* Principal sources of Gaussian noise in digital images arise during acquisition e.g. sensor noise caused by poor illumination and/or high temperature, and/or transmission e.g. electronic circuit noise. 
> .
* In digital image processing Gaussian noise can be reduced using a spatial filter, though when smoothing an image, an undesirable outcome may result in the blurring of fine-scaled image edges and details because they also correspond to blocked high frequencies. 
> .
* Conventional spatial filtering techniques for noise removal include: **mean (convolution) filtering, median filtering and Gaussian smoothing.**
> .
We should specify the width and height of kernel which should be positive and odd. We also should specify the standard deviation in X and Y direction, sigmaX and sigmaY respectively. If only sigmaX is specified, sigmaY is taken as same as sigmaX. If both are given as zeros, they are calculated from kernel size. Gaussian blurring is highly effective in removing gaussian noise from the image.
  
 <p align="center"> 
<img src="https://docs.opencv.org/3.1.0/gaussian.jpg">
</p>
 ---
 
 ##  6.       hsv = cv2.cvtColor(frame2, cv2.COLOR_BGR2HSV)

HSV (hue, saturation, value) colorspace is a model to represent the colorspace similar to the RGB color model. Since the hue channel models the color type, it is very useful in image processing tasks that need to segment objects based on its color. 

Variation of the saturation goes from unsaturated to represent shades of gray and fully saturated (no white component). Value channel describes the brightness or the intensity of the color. Next image shows the HSV cylinder.

<p align="center"> 
<img src="https://docs.opencv.org/3.4/Threshold_inRange_HSV_colorspace.jpg">
</p>

Since colors in the RGB colorspace are coded using the three channels, it is more difficult to segment an object in the image based on its color.

<p align="center"> 
<img src="https://docs.opencv.org/3.4/Threshold_inRange_RGB_colorspace.jpg">
</p>

**HSV** can help you actually pinpoint a more specific color, based on hue and saturation ranges, with a variance of value, for example. 

If you wanted, you could actually produce filters based on BGR values, but this would be a bit more difficult. If you're having a hard time visualizing HSV, don't feel silly, check out the Wikipedia page on HSV, there is a very useful graphic there for you to visualize it. 

Hue for color, saturation for the strength of the color, and value for light is how I would best describe it personally. 
So, After capturing the live stream frame by frame we are converting each frame in BGR color space(the default one) to HSV color space. 

There are more than 150 color-space conversion methods available in OpenCV. 

But we will look into only two which are most widely used ones, BGR to Gray and BGR to HSV. For color conversion, we use the function cv2.cvtColor(input_image, flag) where flag determines the type of conversion. For BGR to HSV, we use the flag cv2.COLOR_BGR2HSV. 

Now we know how to convert BGR image to HSV, we can use this to extract a colored object. In HSV, it is more easier to represent a color than RGB color-space.

In specifying the range , we have specified the range of blue color. Whereas you can enter the range of any colour you wish.

---

## 7. mask = cv2.inRange(hsv, lowergreen, uppergreen)

### Checks if array elements lie between the elements of two other arrays.

* **C++**: void inRange(InputArray src, InputArray lowerb, InputArray upperb, OutputArray dst)

* **Python**: cv2.inRange(src, lowerb, upperb[, dst]) → dst

* **C**: void cvInRange(const CvArr* src, const CvArr* lower, const CvArr* upper, CvArr* dst)

* **C**: void cvInRangeS(const CvArr* src, CvScalar lower, CvScalar upper, CvArr* dst)

* **Python**: cv.InRange(src, lower, upper, dst) → None

* **Python**: cv.InRangeS(src, lower, upper, dst) → None

###Parameters:	

* src – first input array.
* lowerb – inclusive lower boundary array or a scalar.
* upperb – inclusive upper boundary array or a scalar.
* dst – output array of the same size as src and CV_8U type.

* The function checks the range as follows:

..* For every element of a single-channel input array:

<p align="center"> 
<img src="https://docs.opencv.org/2.4/_images/math/daa7c2be6be83bad04d6ef7cfff4b2a7b8a43b52.png">
</p>

..* For two-channel arrays:

<p align="center"> 
<img src="https://docs.opencv.org/2.4/_images/math/3de50754e0df6fd0dac51df248e783b152083cef.png">
</p>
That is, dst (I) is set to 255 (all 1 -bits) if src (I) is within the specified 1D, 2D, 3D, ... box and 0 otherwise.

When the lower and/or upper boundary parameters are scalars, the indexes (I) at lowerb and upperb in the above formulas should be omitted.




