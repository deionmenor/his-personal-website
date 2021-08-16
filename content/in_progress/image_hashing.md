+++
title = "Image Hashing in Python"
description = ""
date = "2021-08-16"
+++


The recent controversy around Apple's decision to use hashing algorithms on the device itself to look for CSAM images has made me interested in looking into how these algorithms work.

I've used websites like TinEye and Google's own reverse image search for my own purposes, stuff like finding higher resolution versions of images to add to my work, or to find the original source of an image. Though these websites are not open-source, its likely that they use some version if the technique known as hashing.

In my former company at Shopee, our team was tasked to look into looking at whether a certain pair of products or SKUs are similar. We devised a multi-step(?) pipeline for flagging such items using features such as product name, price, and category. We also considered looking into product image similarity, but the hashing algorithm set up at the time did not scale well with the volume needed for checking.  

This blog post will go over a variation of hashing dubbed the "dhash" or a difference hash

## How does it work

First thing we need to do is reduce the size of the image. For this example, images will be reduced to a 9pxl by 8 pxl jpg. Afterwards, I'll convert the image to greyscale. 

At this point, the image is basically a 2d array of color values  (0-255). Even at this small size, there is a possibility of 256^72 possible images. The issue however is that the values would not do well when dealing with possible retouches like increasing lightness, contrast, etc. There are several ways to go about this, but for this example, we'll be looking at the relative changes in brightness from one pixel to another.

Computing the difference of the array should give us an 8x8 grid of differences. To create the hash, we can output 1 if the pixel is brighter than the pixel to its left and 0 if otherwise. This should give us a 64bit hash that we can compare with other images.

### Run it in Python

``` python
from PIL import Image

import numpy as np

  

def hammingDist(str1, str2):

 count = 0

 for i in range(0,len(str1)-1):

 if str1[i] != str2[i]:

 count += 1

 return count

  

def get_hash(filename):

 image = Image.open(filename)

  

 # rescale to a 9x8 image

 image = image.resize([9,8], Image.ANTIALIAS)

  

 # convert to greyscale

 image = image.convert("L")

  

 # get difference of the array

 pix = np.array(image)

 dif = np.diff(pix)

  

 im = Image.fromarray(dif)

 im.save("diff/{}_diff.jpg".format(filename))

  

 bits = []

 for i in range(0,7):

 for j in range(0,6):

 bits.append(int(dif[i][j] < dif[i][j+1]))

  

 output = ''.join([str(bit) for bit in bits])

  

 return output

  

original_img = "axolotl.jpg"

original_hash = get_hash(original_img)

print("Original image has the hash: {}".format(hex(int(original_hash, 2))))

  

comparison_images = ["axolotl_small.jpg","axolotl_sepia.jpg","axolotl_bw.jpg","axolotl_2.jpg"]

  

for i in comparison_images:

 hash = get_hash(i)

 ham = hammingDist(original_hash,hash)

 print("{} returns the hash: {} [diff:{}]".format(i,hex(int(hash, 2)),ham))
```

# results
#todo


https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html 