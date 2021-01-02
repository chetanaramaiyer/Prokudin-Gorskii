# Prokudin-Gorskii
Colorizing the Prokudin-Gorskii photo collection

#### In this project, glass plate negatives taken by photographer Sergei Mikhailovich Produkin-Gorskii are aligned to create colorized images.

## Usage

#### To output all of the colored images, you should simply run all the cells in the jupyter notebook file main.ipynb. It runs through all the images in the /data folder and outputs the colored images for each of them.

## General logic
## 1. Smaller .jpg images:
#### For the smaller jpg images, I first crop the green image by 10 percent. Then I align this cropped image against the full red image. By "align", I mean I iteratively go through each possible place that the cropped green image can be placed on the full red image. With each starting point, I calculate the mse for the cropped green image against the corresponding overlapping section of full red image. We return the overlappin section of the full red image for whichever alignment produces the lowest mse. Then, I repeat this process with the cropped green image with the full blue image. Using the cropped green image and the returned red and blue overlapping sections, we create a colored image.

## 2. Bigger .tif images:
#### For the bigger .tif images, I use the pyramid image layering strategy. This pyramid image layering strategy involves using a scaling factor to reduce the  pixel size of the image in each layer. The bottom layer of each pyramid is the original image with the original pixel size. As you go up the pyramid, I use a scaling factor of 2 to reduce the the pixel size of the image in each layer until I get to an image that has <= 50 pixels in width and height.

#### First, I crop the green image by ten percent. Then, I generate the pyramids for each of the red, blue and cropped green images. I align the cropped green layers against the red layers. This alignment involves first running the logic for small .jpg images on the top layer (the image with <= 50 pixels) to compute a general direction of where a good starting point is. Using that initial starting point, we then create a range of where the starting point for the next layer could be. For example if the starting point with the lowest MSE is at (1,1) in top layer, then the starting point could be between (0,0) and (2,2) in the next layer because I used a scaling factor of 2. Instead of computing the mse for all the starting points, we instead compute it for the starting points within that range, which saves a lot of running time. We continune this process until we get to the bottom layer, which contains the original image.

## Functions
### 1. mse(image1, image2)
#### This function takes in 2 images as parameters and computes the mse for those images. The mse is essentially the average of the squared sum difference of each pixel. This function helps us with alignment because it gives us a metric to evaluate which starting points are better than others.

## 2. align_small(image1, image2)
#### This functiton takes in 2 images as parameters. In terms of "alignment", image 2 is always the bigger image and image1 is the cropped image. Therefore, we simply iterate over all the possible starting points for the image1 to be placed on image2. At each starting point, we compute the mse and maintain which starting point results in the alignment with the lowest mse. This function returns the minimum mse value, the cropped section of image2 that results in the alignment with the lowest mse and the starting x,y points that resulted in the alignemnt with the lowest mse.

## 3. crop_ten(image)
#### This function takes in an image as a parameter. It crops ten percent of the image's width and height and returns that cropped image.

## 4. small_jpg_images(r,b,g)
#### This function takes in the separate r,b,g channels that are computed from tthe single large image. We first crop the green image and then we call align_small on the cropped green and blue to get the correct alignment of the blue image. We do the same for the green cropped and the red image. We return the correct alignment of the red and blue image, and the cropped green image.

## 5. align_big(other_layers, base_layers)
#### This function takes in two parameters: the layers for the "base image"(which is just the layers for the green cropped image) and the layers for the "other images" (which is the layers for the red or blue image). We first call align_small on the highest layer of the base and other layers, which is the image with <=50 pixels. This generates the best starting points for the highest layer, which we use to set the range of pixels we should search to find the best starting points for the next layer. We stop once we get to the lowest layer, which contains the original image. This function returns the section of the red/blue image that aligns the best with the cropped green image.

## 6. bigger_image(r,b,g)
#### This function takes in the separate r,b,g channels that are computed from tthe single large image. We first crop the green image. Then, we generate the pyramid layers for cropped green image, the red and blue image. We store those layers in three separate lists. Then, we align the red layers with the cropped green layers to find the correct alignment of the red image. We do the same to find the correct alignment of the blue image. We return the correct alignment of the red and blue image, and the cropped green image.

## 7. run_everything(imname)
#### This function takes in the name of the image and reads in the image, separates it into color channels. If the image name ends with .jpg, I call small_jpg_images and pass in the three channels. If the image name ends with .tif, I call bigger_images and pass in the three channels. Using the results, I display the images.

## 8. main()
#### This reads all the files in the input path (which is the data folder containing all the images) and calls run_everything on each image.
