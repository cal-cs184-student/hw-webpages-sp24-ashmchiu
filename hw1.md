---
layout: page
title: 'Homework 1: Rasterizer'
has_right_toc: true
---
<p class="warning-message">
This assignment has not been completed yet.
</p>

<script>
    window.addEventListener('DOMContentLoaded', function () {
      const container = document.querySelector('.content-nav');
      const navLinks = container.querySelectorAll('.nav-list-link');

      function highlightNavLink() {
        let fromTop = container.scrollTop + 100;

        navLinks.forEach(link => {
          let target = document.querySelector(link.hash);
          if (target.offsetTop <= fromTop && target.offsetTop + target.offsetHeight > fromTop) {
            navLinks.forEach(link => link.parentElement.classList.remove('active'));
            link.parentElement.classList.add('active');
          }
        });
      }

      container.addEventListener('scroll', highlightNavLink);
      highlightNavLink();
    });
</script>

site: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw1/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw1/)

## Overview
TODO

## Task 1: Drawing Single-Color Triangles
Our process of rasterizing triangles involved (1) resolving winding order, (2) calculating minimum and maximum x and y values, (3) checking for each point within bounds whether it would be within the triangle or not.

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
Building off our work in Task 1, we'll walk through how we updated `rasterize_triangle` to supersample. The intention here to supersample is to increase the sampling rate within one pixel (as Task 1 sampled once per pixel) and this serves to decrease aliasing/moire and blend images to be smoother. Our methodology for antialiasing by supersampling involved (1) resizing the sample buffer to account for "over" sampling (sampling multiple times per pixel), (2) perform the three line test for each individual sample similarly to Task 1, and then (3) downsample the `sample_buffer`.

### Resizing the Sample Buffer
Because we want to store more data than there are pixels (since we're sampling multiple times per pixel), we resize the data structure `sample_buffer`. Namely, we know the size of `sample_buffer` begins as `width * height`, so in `set_sample_rate` and `set_framebuffer_target`, we update this value to be `width * height * floor(sqrt(sample_rate)) * floor(sqrt(sample_rate))` in order to store for each individual sample we're taking now, there is an index for it in the `sample_buffer`.

### Three Line Test
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

We can use the image above on the right (`basic/triangle.svg`) as an example. We can see that its bottom left corner is red, its top is green, and its bottom right is blue. In the actual implementation of barycentric coordinates, we know that we still perform the sampling of the previous tasks (namely, task 1-2) where we must sample to see whether points are within a triangle using the three line test. Then, we calculate, for each point within the triangle, its corresponding alpha, beta, and gamma values based on the calculations given during lecture. As such, we then put into the `sample_buffer` the value
{% highlight js %}
alpha * c0 + beta * c1 + gamma * c2
{% endhighlight %}

where `c0`, `c1`, and `c2` are the corresponding colors at the vertices `(x0, y0)`, `(x1, y1)`, and `(x2, y2)`. This is the final color that will be used at that specific pixel. If supersampling is used, we expand again, like in Task 2, to allow further gradients.

On a high level, with a sample rate of 1, we use the vertices as references, and within the triangle, we calculate the proportional distances at each pixel from each vertex. Then, we interpolate, noting that those proportions are what determine how much of each vertex's color is used to render each pixel at. We can see this also in the image `basic/test7.svg`, with many triangles rendering with a color one end and a dark blacker version on the opposing corner and using barycentric coordinates to interpolate and mix the colors of pixels in between.

## Task 5: "Pixel sampling" for texture mapping
TODO

## Task 6: "Level sampling" with mipmaps for texture mapping
TODO

### Images
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task6_1.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_2.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw1/task6_3.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_4.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
  </table>
</div>

From the above four images, we can see from comparing the top row to the second row, utilizing `L_NEAREST` over `L_ZERO` creates a smoother image and comparing the first column to the second column, we can see that using `P_LINEAR` over `P_NEAREST` creates a smoother image as well. Therefore, the best one is `L_NEAREST` adn `P_LINEAR`. Particularly, we compare the jagged edge of the top circular part of the space needle and see that the straight lines are smoother.

Looking for straight lines
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw1/task6_5.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_6.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_ZERO</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw1/task6_7.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_NEAREST</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw1/task6_8.png" width="400px"/>
        <figcaption><code class="language-plaintext highlighter-rouge">L_NEAREST</code> and <code class="language-plaintext highlighter-rouge">P_LINEAR</code></figcaption>
      </td>
    </tr>
  </table>
</div>

These four comparisons corroborate what we discovered above. Particularly, looking at the straight lines, we can see that the smoothest straight lines come from `L_NEAREST` and `P_LINEAR` as the other three options are significantly more jagged, particularly `L_NEAREST` and `P_NEAREST`, having jagged, wavy lines. `L_ZERO` pictures (for both `P_NEAREST` and `P_LINEAR`) are just more messy and pixelated where lines aren't as clear.

## Contributors
Ashley Chiu, Angel Aldaco