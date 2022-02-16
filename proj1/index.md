# CS 184 Assignment 1 Writeup
Arushi Somani & Crystal Wang

## Overview

At a high level, we implemented a basic triangle rasterizer to render svg images, with extensions that allowed for antialiasing via supersampling / pixel sampling / level sampling, as well as color and texture interpolation with barycentric coordinates.

This was a really cool project, especially as an introduction to graphics! We definitely appreciated how it directly applied concepts from lecture - in fact, we felt that our understanding of concepts like mipmaps and bilinear sampling were significantly deepened from doing this assignment. It was also a really interesting experience translating the overarching ideas we learned into actual c++ code. We also really enjoyed poking around at the underlying representations of svg files - we had no idea what they looked like under the hood until we had to create and edit our own! 


## Task 1 

We rasterize triangles by following by iterating through each pixel that falls within the bounds of the three lines of the triangle with vertices at the given (x, y) pairs and filling that pixel with the provided color. The algorithm we used just checks each sample within the bounding box of the triangle, with nested for loops that run from the minimum to maximum given vertex x and y values, which is no worse than / equivalent to one that checks each sample within the bounding box of the triangle.

![basic/test4.svg, with the pixel inspector on a skinny part of the red triangle. I found this interesting because the jaggies are very evident!](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644958112027_screenshot_2-15_12-45-27.png)



## Task 2 

For our supersampling algorithm, we added a nested for loop inside the loops from task 1. We calculate the horizontal and vertical offsets corresponding to the sampling rate for each pixel in the bounding box of the triangle, and sample the color at each of those offsets inside the pixel. After sampling, we update the expanded sample buffer (whose size we updated to be `width * height * sample rate`). We created no new data structures, just modified existing ones (ie. increased size of sample buffer).

Supersampling is particularly useful for antialiasing, because it essentially gets a better heuristic in finer grain to soften and blur edges between shapes so that the cutoffs are less sharp and jagged. We used supersampling to antialias our triangles by taking more samples per pixel within the bounding box of the triangle to smooth out transitions in colors on the screen, which particularly helped the smooth sharp edges and corners in the images.

In the process of adding a supersampling algorithm to our code, we had to tweak the rasterization pipeline a bit. When actually putting rgb values into the frame buffer, we first had to average the rgb values for the `sample_rate` samples for each pixel, then actually pass in those values to the frame buffer. Because we had to expand our sample buffer size to accommodate all the samples we took for each pixel, we reoriented the indexing of the sample buffer so that it was ordered in the manner of `[pixel 1 sample 1, pixel 1 sample 2, … , pixel n sample m]`.

![basic/test4.svg, sample rate 1](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644958146878_screenshot_2-15_12-45-42.png)
![basic/test4.svg, sample rate 4](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644958146885_screenshot_2-15_12-45-54.png)

![basic/test4.svg, sample rate 16](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644958146897_screenshot_2-15_12-46-6.png)

    We observe these resulting differences from the three images with different sample rates because the more samples we take per pixel, the better the final rendered color will blend into the surrounding pixels and the image’s edges will blur out. When we only sample once per pixel, there is clearly no gradual attempt to blend colors (as seen in the pixel inspector), so the corner of the triangle appears very jagged and even discontinuous. On the other hand, for 4 and 16 samples per pixel, we see pixels in the pixel inspector that take on colors between red and white, which allow for a “softer” appearance and more natural edges/lines than just singular sampling can achieve.


## Task 3 

**Air traffic controller who’s blasting jams through their noise-cancelling headphones**: I was trying to angle the left arm up in a waving position and have the right leg kick back at an angle.

![](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644959234402_screenshot_2-15_13-6-58.png)



## Task 4

Barycentric coordinates are a coordinate system in which any point in two-dimensional space is described as the weighted combination of the vertices of some given triangle. These weights are known as `alpha`, `beta` and `gamma` values of the given coordinate. The sum of these coordinate weights is always 1.


![a triangle with the vertices being colored red, blue and green.](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644967235933_screenshot_2-15_15-20-26.png)

![svg/basic/test7.svg with default viewing parameters and sample rate 1](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644959340731_screenshot_2-15_13-8-40.png)



## Task 5

Pixel sampling is, stated coarsely, the concept of getting Color (rgb) values for certain pixel locations of a texture. To get the actual pixel value of a texture is not a complicated process — one just has to return the `texel` value of that location. However, pixel sampling has different methods, and we shall discuss two of them here: nearest neighbor and bilinear sampling. The idea of nearest neighbor is to return the color of the nearest `texel` to the given location. The idea of bilinear sampling is to get the color of the four nearest pixels, then return the `texel` representing their weighted average. One implementation detail to mention here: The passed-in argument `uv` is technically in the range of [0, 1] – it’s a weight, not an actual measure of the location on the texture space, and must therefore be scaled by the height and width of the texture in both cases.

![Nearest sampling](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644964924895_screenshot_2-15_14-35-5.png)
![Bilinear sampling](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644964924914_screenshot_2-15_14-41-14.png)



![Nearest sampling, 1 sample/pixel](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644965020032_screenshot_2-15_14-43-1.png)
![Nearest sampling, 16 samples/pixel](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644965020055_screenshot_2-15_14-43-16.png)

![Bilinear sampling, 1 sample/pixel](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644965020083_screenshot_2-15_14-43-6.png)
![Bilinear sampling, 16 samples/pixel](https://paper-attachments.dropbox.com/s_FC80D83E19746C4E16342D2040CF0C3495191F5DA077DCA476BE82EB85BF720F_1644965020103_screenshot_2-15_14-43-23.png)


I find that the biggest difference is in when there are sharp lines between two colors — like in the letters in the images above. Notice how the word “Berkeley” looks jagged in nearest sampling, and clearer and smoother in bilinear sampling.


## Task 6 

The idea of level-sampling is that, instead of downsampling from a given texture when we need it, we go ahead and create lower resolution levels of our texture image. These are known as our mipmap levels. In level sampling, we mathematically figure out what level of the map we wish to sample from, and then sample from that level of the mipmap. This was mainly implemented through `Texture::sample` and `Texture::get_level`.  `get_level` is a helper function that returns the level of mipmap we should be using given certain values of `uv`. `sample` is a function that toggles between various level sampling methods, using `get_level` to access the necessary information and returning a sample.

Tradeoff comparisons

- Pixel sampling: Bilinear very good at anti-aliasing, nearest sampling marginally faster than bilinear
- Level sampling: Linear level-sampling slower than nearest-level as well as level-zero. Any sort of level-sampling better at anti-aliasing than zero-level sampling.
- Number of samples/pixel: Gets substantially slower as this increases, and definitely keeps getting more and more memory intensive. Antialiasing power also increases.



![Zero Level, Nearest Pixel Sampling](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644969778613_screenshot_2-15_16-0-50.png)
![Zero Level, Bilinear Pixel Sampling](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644969811807_screenshot_2-15_16-0-54.png)

![Nearest Level, Nearest Pixel Sampling](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644969822372_screenshot_2-15_16-1-8.png)
![Bilinear Level Interpolation, Nearest Pixel Sampling](https://paper-attachments.dropbox.com/s_31E418C4698438C0F670A15712DF85A5C1A9B10DA0219BC3DD8DB1B1EAFA3459_1644969837791_screenshot_2-15_16-1-15.png)


