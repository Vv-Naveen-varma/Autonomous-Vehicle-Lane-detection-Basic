# Lane Detection Theory
## Steps and methods involved
### Step-1: Convert the image to GrayScale
As we know, images are made up of pixels, our input colored image has 3 channels (Red, Green, Blue) here each pixel is a combination of three intensities. Where as a Grey-Scale image only has 1 channel each pixel with only 1 intensity value ranging from 0 to 255. So, by using a GrayScale image processing a single channel is faster than processing a three channel color image.
### Step-2: Reduce noise in an image
* When detecting edges, we must filter-out any image noise. Image noise can create false edges and ultimately effect edge detection, that's why its necessary to filter the noise and thus smoothen the image.
* This process is implemented using Gaussian Blur,"How do you do it?". As we know image is stored as a collection of pixels,  each pixel is represented by a single number that discribes the brightness of a pixel. We modify the value of a pixel with the average values of the pixel intensities around it. Averaging out the pixels will be done with a kernel, essentially this kernel of normally distributed numbers is run across our entire image and sets each pixel value equal to the average of its neighbouring image.
                        blur = cv2.GaussianBlur(gray, (5,5), 0)
      This indicates that we are applying a gaussian blur on grayScale image with a 5 by 5 kernel, the size of the kernel depends on specific situation. This returns a smoothed image (reduced noise)

### Step-3: Apply Canny method to identify edges in our image
* An edge corresponds to a region where there is a sharp change in intensity or color between adjacent pixels in the image.Pixels in the image can be read as a matrix. The change in brightness over a series of pixels is called "gradient". A strong gradient (example:[0 0 255 255]) indicates great change where as small gradient (example:[0 0 20 20]) shallow change. We can represent this in a graph.

![graph](https://user-images.githubusercontent.com/63995834/121150704-61048080-c861-11eb-9591-990f0cb01f24.jpg)
* what a canny function does? It measures adjacent changes in intensity in all directions, x and y. It produces a Gradient image by tracing the edge with large change in intensity (large gradient) in an outline of white pixels. The ratio recommended to use between low and high thresholds is 1:3 or 1:2 by the documentation

![canny](https://user-images.githubusercontent.com/63995834/121150865-84c7c680-c861-11eb-8869-4c8c27dda0b1.jpg)

### step-4: Find the region of interest by finding the vertices, using matplotlib, and forming a triangle
* Let us consider a sample image, which looks like this after applying the canny function

![sample-canny](https://user-images.githubusercontent.com/63995834/121151499-0cadd080-c862-11eb-9141-600c28473923.jpg)
* Now we need to find the area of interest from the above image, "how do you get this area of interest?". we plot the image using pyplot from matplotlib library and note the vertices of the triangle region where the required lane region is present. If you observe the image below we can find a triangle with vertices left_lane_start=100 right_lane_start=860 left_lane_end=470 right_lane_end=300. We consider that the position of the camera is stable in our system so all the images captured have same vertices the region of interest forming a tirangle.  

![canny-graph](https://user-images.githubusercontent.com/63995834/121151397-f0aa2f00-c861-11eb-817e-895e6cd9fa6a.jpg)

* After the canny_image with region of interest, we mask the image by performming a Bitwise_and operation on the canny_image and mask_image (which consists of zeros as its pixel values) which results in filtering the unnecessary region from the image and outputs the below image.

![region-of-interest](https://user-images.githubusercontent.com/63995834/121151115-b93b8280-c861-11eb-9490-0c30337d3e65.jpg)

### step-5: Hough Transform:
    What is hough transform you ask?
* y = mx+b  [Stright line equation with parameters m and b]

    b = y-intercept

    m = (change in y) / (change in x)

    image space is a fucntion of plot x and y

    hough space is a function of plot m and b

    Note that a single point in image space is represented by a line in hough space, as shown below

![hough-sample](https://user-images.githubusercontent.com/63995834/121151208-cb1d2580-c861-11eb-9abf-6e39b15e787f.jpg)
* Consider a sample image space and its corresponding hough space as shown below,

![sample-hough](https://user-images.githubusercontent.com/63995834/121151614-27804500-c862-11eb-868b-cb721d20eb83.jpg)
* we first split our hough space into a grid (bins), each bin corresponding to the slope and y-intercept value of a candidate line. For every point of interception we are going to cast a vote inside of the bin it belongs to.

![hough-bins](https://user-images.githubusercontent.com/63995834/121151642-2f3fe980-c862-11eb-92ad-60bb0b8538e5.jpg)
* The bin with the maxium number of votes is going to be our line, what ever the m and b value this bin belongs to that's the line we are going to draw as it is the best fit in discribing our data.

![hough-selected](https://user-images.githubusercontent.com/63995834/121151690-38c95180-c862-11eb-806d-309b349f12ad.jpg)
* But there is an error associated with this model. If you calculate the slope of a vertical line you get "infinity", which is a big "NO".

![slope-infinity](https://user-images.githubusercontent.com/63995834/121151732-3ff05f80-c862-11eb-8b89-d10a4ecdeed6.jpg)
* To avoid these types of numerical problems, we use polar coordinate system, i.e, row and data

                    p = xcos(@) + ysin(@) where p = row and @ = theta 

        p = xcos(@) + ysin(@) is a line equation in polar coordinates

        where p = perpendicular distance from the origin and @ = angle of inclination of normal line from x-axis

* Now, with polar coordinates for a given point in image space we get a curve in hough space that is a plot of p and @.This curve represents all the lines passing through that point in image space.

![curve](https://user-images.githubusercontent.com/63995834/121151794-4aaaf480-c862-11eb-85de-c97e3b85e377.jpg)
* Now the concept is same as before, we find the bin with max interceptions of curevs.

![final](https://user-images.githubusercontent.com/63995834/121151816-51396c00-c862-11eb-9bc7-dd3a98ba1994.jpg)
### Finally
* You finally made it. These are all the concepts that are required in this program. Now, have a look at the code and check your knowledge
