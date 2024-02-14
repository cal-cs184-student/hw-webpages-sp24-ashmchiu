---
layout: page
title: 'Homework 1: Rasterizer'
has_right_toc: true
---

site: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw1/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw1/)

## Overview
Starting off by rasterizing single-color triangles, this homework developed into a deeper conversation regarding supersampling and aliasing and sampling techniques when it comes to texture mapping. Texture mapping was an interesting development in understanding how we could modify not only the number of samples we take per pixel to render (mapping surface space to texture space), but also at what depth we render each pixel, choosing locations to render with lower detail to antialias. With a focus on different strategies to reduce aliasing, we learned of different strategies, using barycentric coordinates and linear interpolation to further smooth out jaggies in the resulting graphics. Not only this, we also were tasked with understanding how using these different tools that modify the rasterization pipeline affected speed and memory usage. Finally, we also were tasked to transform a simple, but humble cubeman using matrix trasnformations and homogenous coordinates into someone completely new (slay cubeman)!

Some interesting things we learned when completing this homework were:
- Rasterization as a whole involves a lot more than just pixel manipulation: throughout the project, understanding how surface space mapped to texture space and back was an intriguing process to go through.
- Seeing the comparisons of combinations of different pixel and level sampling techniques was cool to see. The idea of mipmaps is still a fun concept to wrap our heads around in the ways that it can target different aspects of images in different ways (in order to avoid aliasing). 
- How to work with `.svg` files, namely how to read their values and modify them. Understanding how to manipulate file types was something we've never attempted before, so that was definitely a learning curve!


## Task 1: Drawing Single-Color Triangles
Our process of rasterizing triangles involved 
1. resolving winding order, 
2. calculating minimum and maximum x and y values,
3. checking for each point within bounds whether it would be within the triangle or not.

### Resolving Winding Order
To ensure that all triangles rendered (particularly for `test6.svg`), we ensured that all points were in counterclockwise order. Particularly, if this wasn't the case, we would swap `(x1, y1)` and `(x2, y2)`. This is important to ensure that regardless of how the triangles are given to be rasterized, when we later check whether they should be colored or not (via the three line test), this ensured that we were always checking that the points were within the lines of the triangle, not sometimes outside.

### Calculating Minimums and Maximums
We calculated the minimum and maximum `x` and `y` values encompassed by our triangle. Namely, we took the overall minimum `x` across the three vertices, the minimum `y` across the three vertices, the maximum `x` across the three vertices, and the maximum `y` across the three vertices. This is important since we're essentially creating a rectangle of space that spans from `(minX, minY)` to `(maxX, maxY)`. This ensure that the algorithm is no worse than one that checks each sample within the bounding box of the triangle because we are in effect, calculating the boudning box of the triangle, of which the vertices are `(minX, minY)`, `(minX, maxY)`, `(maxX, minY)`, and `(maxX, maxY)`. As we only sample within this box (in a double loop), we've met the necessary criteria.

### Sampling
Finally, for each point within the bounding box outlined above, we would check for the three lines dictated by the edges of the triangle, whether the point was within the triangle or not. Our standardization here was checking the the formula for the line test, and whether 
{% highlight js %}
(-(x - xi) * dYi) + ((y - yi) * dXi)
{% endhighlight %}
was greater than or equal to 0. If so, then we would fill that pixel.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task1.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with default viewing parameters</figcaption>
      </td>
    </tr>
  </table>
</div>

Above, we've included a screenshot of `basic/test4.svg` with the default viewing parameters and the pixel inspector centered on an interesting part of the scene. Particularly, we can see the jaggies on this rasterized corner of the blue triangle.

## Task 2: Antialiasing by Supersampling
Building off our work in Task 1, we'll walk through how we updated `rasterize_triangle` to supersample. The intention here to supersample is to increase the sampling rate within one pixel (as Task 1 sampled once per pixel) and this serves to decrease aliasing/moire and blend images to be smoother. Our methodology for antialiasing by supersampling involved
1. resizing the sample buffer to account for "over" sampling (sampling multiple times per pixel) and performomg the three line test for each individual sample similarly to Task 1, and then,
3. downsample the `sample_buffer`.

### Resizing the Sample Buffer
Because we want to store more data than there are pixels (since we're sampling multiple times per pixel), we resize the data structure `sample_buffer`. Namely, we know the size of `sample_buffer` begins as `width * height`, so in `set_sample_rate` and `set_framebuffer_target`, we update this value to be `width * height * floor(sqrt(sample_rate)) * floor(sqrt(sample_rate))` in order to store for each individual sample we're taking now, there is an index for it in the `sample_buffer`.

Then, using work from Task 1 as a baseline, we kept the double while loop to traverse through each pixel within the bounding box of the triangle. However, for each individual pixel, we mark that we will sample points `1 / sqrt(sample_rate)` spaced apart. Something that we also had to account for here, was ensuring that the first sampled point was actually `0.5 / (sample_rate)` away from the edge so we're sampling the center of each mini pixel we're essenitally creating. Thus, within each individual pixel that we were sampling from Task 1, we now sampling `sqrt(sample_rate)` times in the `x` direction per `sqrt(sample_rate)` times in the `y` direction. Then, we sample each of these points, running the three line test on them, and add them to the resized sample buffer if they lie within the triangle.

Note here, we had to perform calculations to index the `sample_buffer`. The index of a sampled point `x, y` within the sample buffer would be
{% highlight js %}
(((y * sample_size + dy) * width) * sample_size) + (x * sample_size + dx)
{% endhighlight %}

where `sample_size = floor(sqrt(sample_rate))`, and `dx` and `dy` represent the distance from the sampled point to the left edge of the pixel and the bottom edge of the pixel respectively.

### Downsampling
Finally, since we sampled `sqrt(sample_rate) * sqrt(sample_rate)` times per pixel, we now needed to downsample. To do so, we average all the `sqrt(sample_rate) * sqrt(sample_rate)` samples we took across the pixel and sent that to the frame buffer. This work was done in `resolve_to_framebuffer`. We also had to make modifications to `fill_pixel` to ensure that rendered points and lines used our same resized `sample_buffer` indices.

### Updates to Rasterization Pipeline
- `rasterize_triangle`: instead of sampling once within a pixel, we sample at `sqrt(sample_rate) * sqrt(sample_rate)` equally spaced points within a pixel
- `resolve_to_framebuffer`: as we resized the `sample_buffer` to allow for the supersampling, `resolve_to_framebuffer` did the work to downsample and produce one color per pixel (gradient depending on how many samples within a pixel was within the triangle).
- `fill_pixel`: because of the adjacent change to the `sample_buffer`, we had to ensure that points and lines also followed the indexing pattern of the `sample_buffer`.

### Impacts of Supersampling
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task2_1.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with sample rate of 1</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task2_2.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with sample rate of 4</figcaption>
      </td>
    </tr>
    <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw1/task2_3.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with sample rate of 16</figcaption>
      </td>
    </tr>
  </table>
</div>

We can see here through the images that as the sample rate increases, the right corner of the red triangle gets more blurred out and has less jaggies/aliasing. In the image with a sample rate of 1, the staircase jaggies are prevelant without even zooming in, and the red pixels zoomed in aren't even touching. In the image with a sample rate of 4, we can still see jaggies rendered in the image, but they're not as prominent. In the pixel inspector, we can see gradients of red (not just solid red), that signify points where some, but not all, of the sampled points within the pixel were in the triangle. Finaally, in the image with a sample rate of 16, the gradient in the pixel inspector creates an even smoother corner within the actual image itself, where now to see the jaggies, I have to look a lot harder.

Instead of only sampling once per pixel, by supersampling, we don't have a binary flip of whether a pixel is inside the triangle or not, but rather a gradient. For instance, for a sample rate of 4, the blur means that we could represent pixels with only 3 sample points within the triangle as lighter (3/4) than those which were fully within the triangle (all 4 sample points were within the triangle). This blurs edges and when zoomed out, ensures that images actually look cleaner (antialiasing the triangles). As such, supersampling is useful because as we can see in the image itself, the triangle corner is cleaner (there are no staircase patterns) as the sampling rate increases. 

## Task 3: Transforms

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task3.png" width="400px"/>
        <figcaption>Rasterizing transforms/my_robot.svg with default viewing parameters</figcaption>
      </td>
    </tr>
  </table>
</div>

Here, we implemented the `translate`, `scale`, and `rotate` methods with matrices that held homogenous coordinates. We included our rendition of `my_robot.svg` above, performing extra rotations and transforms.
Particularly,
- the legs included rotations and translations to move them into a bent, sitting down formation (like he's meditating)
- the arms included rotations and translations to move them into an arms up stance (similar to one where you're extremeley happy and excited).

Cubeman is happily resting (meditating, yet joyful), hoping that you are also happy and getting good rest as well! <3

## Task 4: Barycentric coordinates
The purpose of barycentric coordinates is to interpolate across vertices. Namely, an example is calculating whether a point is relative to the vertices of a triangle and where it lies within a triangle using a combination of its weights (dictated by its proportional distances to each vertex of the triangle). For this project, we used barycentric coordinates for sampling and rasterization, such as in the images below and texture mapping for later tasks.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task4_1.png" width="400px"/>
        <figcaption>Rasterizing basic/test7.svg with default viewing parameters, sample rate 1</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task4_2.png" width="400px"/>
        <figcaption>Rasterizing basic/triangle.svg with default viewing parameters, sample rate 1</figcaption>
      </td>
    </tr>
  </table>
</div>

### Triangle Example
We can use the image above on the right (`basic/triangle.svg`) as an example. We can see that its bottom left corner is red, its top is green, and its bottom right is blue. In the actual implementation of barycentric coordinates, we know that we still perform the sampling of the previous tasks (namely, task 1-2) where we must sample to see whether points are within a triangle using the three line test. Then, we calculate, for each point within the triangle, its corresponding alpha, beta, and gamma values based on the calculations given during lecture. As such, we then put into the `sample_buffer` the value
{% highlight js %}
alpha * c0 + beta * c1 + gamma * c2
{% endhighlight %}

where `c0`, `c1`, and `c2` are the corresponding colors at the vertices `(x0, y0)`, `(x1, y1)`, and `(x2, y2)`. This is the final color that will be used at that specific pixel. If supersampling is used, we expand again, like in Task 2, to allow further gradients.

On a high level, with a sample rate of 1, we use the vertices as references, and within the triangle, we calculate the proportional distances at each pixel from each vertex. The closer the pixel is to a vertex, the more pull that vertex has in the final decision of that pixels' color. Then, we interpolate, noting that those proportions are what determine how much of each vertex's color is used to render each pixel at. We can see that exactly in the middle of the triangle is a murky, browny color, which represents the situation in which alpha, beta, and gamma are equivalent and thus, the three colors mix with not one holding more influence than any other. We can see this also in the image `basic/test7.svg`, with many triangles rendering with a color one end and a dark blacker version on the opposing corner and using barycentric coordinates to interpolate and mix the colors of pixels in between.

## Task 5: "Pixel sampling" for texture mapping
Given a texture, pixel sampling does the work to map coordinates in `(x, y)` form of the surface space, into the `(u, v)` coordinates of texture space. From here, since  utilizing the texture map's texture at point `(x, y)` for display. The fundamental labor of pixel sampling mimics that of wrapping a chocolate wrapper around a 3D object--it creates a mapping between the wrapper to 3D space. Now, we'll walk through how we implement pixel sampling to perform texture sampling: 
1. for each pixel within the bounding box of the triangle, we check whether it is within the triangle (supersampling if that is what is desired, read [Task 2](/hw1.md#task-2-antialiasing-by-supersampling)),
2. calculating the barycentric coordinates in the `(x, y)` surface space and then applying the formula to determine the correlating `(u, v)` texture space coordinate (read [Task 4](/hw1.md#task-4-barycentric-coordinates)),
3. performing the sampling itself. 

Since we've covered steps (1) and (2) previously, we'll dive deeper into step (3), detailing the implementation strategies of nearest neighbor sampling and bilinear sampling.

### Nearest Neighbor Sampling
For nearest neighbor sampling, we solely use the nearest texture pixel, or texel within the `(u, v)` space as the texture value for the `(x, y)` point. We do this by scaling our `u` and `v` values to the width and height of the mipmap given to us, rounding that value, and then using that as the texture for our pixel.

### Bilinear Sampling
For bilinear sampling, we instead rely not only on the nearest texture pixel, but the four nearest texture pixels. This gives a bit more buffer room since we know that `(x, y)` and `(u, v)` mappings may not be exact, so like with barycentric coordinates, we kind of calculate how much of an influence each of the four texture pixels should get (and that determines the pull of how much weight that textel's texture goes into the pixel's final value). We linearly interpolate three times, using the procedure outlined in lecture, across these four to mix the texture, giving us our final result per pixel. There are two horizontal linear interpolations and one final one on the vertical.

### Campanile

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task5_1.png" width="400px"/>
        <figcaption>Rasterizing texmap/test6.svg, sample rate 1, nearest sampling</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task5_2.png" width="400px"/>
        <figcaption>Rasterizing texmap/test6.svg, sample rate 1, bilinear sampling</figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw1/task5_3.png" width="400px"/>
        <figcaption>Rasterizing texmap/test6.svg, sample rate 16, nearest sampling</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task5_4.png" width="400px"/>
        <figcaption>Rasterizing texmap/test6.svg, sample rate 16, bilinear sampling</figcaption>
      </td>
    </tr>
  </table>
</div>

The differences in these images (particularly when comparing the left-side nearest neighbor sampling to the right-side bilinear sampling) is the fact that bilinear sampling creates smoother, straighter lines. This is clear in the Campanile image as we're looking at the lines of the windows, which are thin. Specifically for the sample rate of 1 per pixel, we note there is heavy aliasing within the nearest neighbor sampling image in the pixel inspector, much more than the bilinear sampling. In the nearest neighbor image, it looks like parts of the thin lines have been carved out. The bilinear sampling image has smoother/fainter aliasing due to the fact that it takes into consideration four neighboring pixels instead of just one. This makes sense, though, because we note that bilinear sampling takes into account 4 pixels and mixes between them while nearest neighbor sampling picks one, which may lead to jaggier/extremeley sharp, contrasting aliasing.

We do note that increasing the sample rate increases the quality (reduces aliasing artifacts) for both nearest neighbor sampling and bilinear sampling, notably more so for nearest neighbor sampling than bilinear sampling. Particularly, the curved arches of the campanile are more visibly curved (and less blocky and staircase-like) in both the nearest neighbor sampling and bilinear sampling images at a sample rate of 16 per pixel over a sample rate of 1 per pixel. The difference in the jump for nearest neighbor sampling is higher.

In general, we see that bilinear sampling does a better job at smoothing out the images (in both cases, the bilinear sampling image does have less jaggies and aliasing, and is smoother overall).

However, it is important to note the benefits of using nearest neighbor sampling as well, over bilinear sampling. Particularly, since bilinear sampling averages across four different values while nearest neighbor sampling only needs one, the computation power needed for nearest neighbor sampling is definitely lower. There will be a large difference between the two methods therefore, in computation power of large images due to the need of performing more than 4 times the computation (not only 4 samples over 1, but also 3 linear interpolations). In regards to the output, we've seen when bilinear sampling would win over nearest neighbor sampling (namely by decreasing aliasing), but due to the fact that bilinear sampling mixes 4 values, there's also a possibility that it may overmix and blend out an image too much. This may cause images to soften (smooth out), but also dull an image. For example, below, we can see that the white on this parrot is dulled out (and although that might be beneficial in blending), some art styles may not prefer that the colors in some high contrast/sharply contrasting areas are sometimes dulled out. The two sampling methods would most likely perform similarly in areas with low contrast (view wise) or few details.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task5_5.png" width="400px"/>
        <figcaption>Rasterizing texmap/test4.svg, sample rate 16, nearest sampling</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task5_6.png" width="400px"/>
        <figcaption>Rasterizing texmap/test4.svg, sample rate 16, bilinear sampling</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 6: "Level sampling" with mipmaps for texture mapping
Finally, our last task was to perform level sampling: utilizing the mipmap mentioned in the last task! Our level sampling built off [Task 5](/hw1.md#task-5-pixel-sampling-for-texture-mapping)'s pixel sampling, with some modifications.

First though, let's discuss what level sampling is and how we implemented it. Conceptually, level sampling allows us to use a mipmap, which dictates at what level we sample, where higher levels signify less detail while lower levels signify more detail (with the most detailed being the base level 0). Level sampling is useful however, since we note that although we are essentially decreasing the detail in higher level sampled areas, we are smoothing out those values and able to address and target aliasing. Specifically, an example of the useful in level sampling is if we have high-contrasting areas that are far away--since that area is smaller and has high contrasting pixel values, there may be a lot of aliasing if we just sampled everything at level 0. Moving that to a higher level allows us to smooth over the high contrast, anti-aliasing it: and it's totally worth it! For something that is meant to be far away, it makes sense for it to have less detail.

However, since high level resolutions would be blurry, we want a method that allows us to interweave high and low level mip maps to textures so the image looks seamless regardless of whether the texture is close or far away. For actual implementation, what we did was build off the work from [Task 5](/hw1.md#task-5-pixel-sampling-for-texture-mapping), and now instead of just calculating the barycentric coordinates for `(x, y)`, we also calculated the barycentric coordinates for `(x + 1, y)` and `(x, y + 1)`, passing those as parameters into our sampling methods. There are three different types of level sampling we implemented: zero level, nearest level, and interpolation (a continuous mipamp). In order to calculate levels for nearest level and interpolation (a continuous mipmap), we wrote a helper function `get_level`, which calculates the difference vectors from the point to the chosen texel, and uses the formula
{% highlight js %}
log2(max(sqrt(dudx^2 + dvdx^2), sqrt(dudy^2 + dvdy^2)))
{% endhighlight %}

to calculate the level.

### Zero Level
When we use the zero level of the mipmap, we're essentially always using level 0 (hardcoded), which represents the original texture. This means we perform no smoothing and was the base that we started off with.

### Nearest Level
When we use the nearest level of the mipmap, for each pixel, we use the helper `get_level` function defined above, clamping the returned level, and then calling the pixel sampling methods defined from [Task 5](/hw1.md#task-5-pixel-sampling-for-texture-mapping), passing in the level we got from `get_level`.

### Interpolation (A Continuous Mipmap)
When we use interpolation (a continuous mipmap), this means that for each pixel, after calling the helper `get_level`, we determine the level above and below and sample at the level above and below, then perform linear interpolation between the two.

### Tradeoffs
We now have the ability to adjust the sampling technique by selecting pixel sampling (nearest neighbor or bilinear), level sampling (zero, nearest, or interpolating), or the number of samples per pixel (1, 4, 9, 16). With these options, we're given a grand total of 24 total different ways to produce a resulting image.

Let's discuss the tradeoffs between speed, memory usage, and antialiasing power between these three techniques.
- Speed: The fastest of the three would most likely be pixel sampling since all we're doing is sampling nearby pixels (at worst) and computing barycentric coordinates. Level sampling can be slower than pixel sampling because even though mipmaps can be precomputed, the actual effort of determining what mipmap level to use (and potentially even interpolating two), will slow down the rendering process. Both pixel sampling and level sampling need to covert data from surface space to texture space. However, these are constant sized increases in operations, summing up to linear in the number of pixels, whereas supersampling would be multiplicative. We see that increasing the number of samples per pixel is definitely slower than pixel sampling since we now have to do extra computations per pixel, namely by sampling it multiple times across the pixel--this is multiplicative in nature and adds up.
- Memory Usage: Pixel sampling shouldn't inherently change the memory usage because we're just converting surface space to texture space, but not requiring extra data structures to do so. Level sampling can take up more memory particularly since we now may need to store more than just one mipmap. However, if these values are precomputed and stored somewhere on disk, this wouldn't require more memory usage than pixel sampling. Increasing the number of samples is the one that increases memory usage the most as we had to resize the `sample_buffer` from its original size to one that houses enough for all the super sampled values within each pixel, again, this is again, multiplicative with respect to an increase in the sampling rate.
- Antialiasing Power: Finally, antialiasing power seems to be slightly difficult to analyze. The simpler case is for supersampling, which is the best for antialiasing since increasing the sample rate and then downsampling with gradients allows us to decrease aliasing. In fact, from the Nyquist Theorem, we know there will be no aliasing if all frequencies are less than the Nyquist frequency, so increasing the sampling rate to twice that of the highest frequency means that there will be no aliasing. On the other hand, pixel sampling's antialiasing power lies in which type of pixel sampling we use: particuarly, bilinear sampling will definitely be better at anti-aliasing than nearest neighbor due to the fact that it samples from four textels instead of one. Level sampling is more likely to help with antialiasing than pixel sampling as it builds off of pixel sampling to help decrease aliasing depending on the depth of the part of an image. Particularly, the interpolating level sampling means that in certain parts of the images, details get blurry, but its due to this blur that we are able to antialias.

### Space Needle
We use these following images to demonstrate the differences in output when pairing different pixel sampling strategies with different level sampling stratgies. Namely, we're testing [nearest neighbor](/hw1.md#nearest-neighbor-sampling) and [bilinear](/hw1.md#bilinear-sampling) pixel sampling and pairing that with [zero](/hw1.md#zero-level) and [nearest](/hw1.md#nearest-level) level sampling.

The following images all use a sample rate of 1 per pixel.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task6_1.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_2.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw1/task6_3.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_4.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
  </table>
</div>

From the above four images, we can see from comparing the top row to the second row, utilizing `L_NEAREST` over `L_ZERO` creates a smoother image and comparing the first column to the second column, we can see that using `P_LINEAR` over `P_NEAREST` creates a smoother image as well. Therefore, the best one is `L_NEAREST` adn `P_LINEAR`. Particularly, we compare the jagged edge of the top circular part of the space needle and see that the straight lines are smoother.

The following images all use a sample rate of 1 per pixel.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task6_5.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_6.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw1/task6_7.png" width="400px"/>
        <figcaption>Rasterizing texmap/seattle.svg, <code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_8.png" width="400px"/>
        <figcaption><Rasterizing texmap/seattle.svg, code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
  </table>
</div>

These four comparisons corroborate what we discovered above. Particularly, looking at the straight lines, we can see that the smoothest straight lines come from `L_NEAREST` and `P_LINEAR` as the other three options are significantly more jagged, particularly `L_NEAREST` and `P_NEAREST`, having jagged, wavy lines. `L_ZERO` pictures (for both `P_NEAREST` and `P_LINEAR`) are just more messy and pixelated where lines aren't as clear.

## Contributors
Ashley Chiu, Angel Aldaco