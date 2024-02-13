---
layout: page
title: 'Homework 1: Rasterizer'
---
<p class="warning-message">
This assignment has not been completed yet.
</p>

site link: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw1/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw1/)

## Overview
TODO

## Task 1: Drawing Single-Color Triangles
Our process of rasterizing triangles involved (1) resolving winding order, (2) calculating minimum and maximum x and y values, (3) checking for each point within bounds whether it would be within the triangle or not.

### Resolving Winding Order
To ensure that all triangles rendered (particularly for `test6.svg`), we ensured that all points were in counterclockwise order. Particularly, if this wasn't the case, we would swap `(x1, y1)` and `(x2, y2)`. This is important to ensure that regardless of how the triangles are given to be rasterized, when we later check whether they should be colored or not (via the three line test), this ensured that we were always checking that the points were within the lines of the triangle, not sometimes outside.

### Calculating Minimums and Maximums
We calculated the minimum and maximum `x` and `y` values encompassed by our triangle. Namely, we took the overall minimum `x` across the three vertices, the minimum `y` across the three vertices, the maximum `x` across the three vertices, and the maximum `y` across the three vertices. This is important since we're essentially creating a rectangle of space that spans from `(minX, minY)` to `(maxX, maxY)`. This ensure that the algorithm is no worse than one that checks each sample within the bounding box of the triangle because we are in effect, calculating the boudning box of the triangle, of which the vertices are `(minX, minY)`, `(minX, maxY)`, `(maxX, minY)`, and `(maxX, maxY)`. As we only sample within this box (in a double loop), we've met the necessary criteria.

### Sampling
Finally, for each point within the bounding box outlined above, we would check for the three lines dictated by the edges of the triangle, whether the point was within the triangle or not. Our standardization here was checking whether the the formula
{% highlight js %}
(-(x - xi) * dYi) + ((y - yi) * dXi)
{% endhighlight %}
was greater than or equal to 0. If so, then we would fill that pixel.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="assets/hw1/task1.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with default viewing parameters</figcaption>
      </td>
    </tr>
  </table>
</div>

Above, we've included a screenshot of `basic/test4.svg` with the default viewing parameters and the pixel inspector centered on an interesting part of the scene. Particularly, we can see the jaggies on this rasterized corner of the blue triangle.

## Task 2: Antialiasing by Supersampling
TODO

### Impacts of Supersampling
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="assets/hw1/task2_1.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with sample rate of 1</figcaption>
      </td>
      <td align="center">
        <img src="assets/hw1/task2_2.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with sample rate of 4</figcaption>
      </td>
    </tr>
    <tr>
      <td colspan="2" align="center">
        <img src="assets/hw1/task2_3.png" width="400px"/>
        <figcaption>Rasterizing basic/test4.svg with sample rate of 16</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 3: Transforms
TODO

## Task 4: Barycentric coordinates
TODO

## Task 5: "Pixel sampling" for texture mapping
TODO

## Task 6: "Level sampling" with mipmaps for texture mapping
TODO

### Authors
Angel Aldaco, Ashley Chiu