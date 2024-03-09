---
layout: page
title: 'Homework 3: Pathtracer'
has_right_toc: true
usemathjax: true
---

<p class="warning-message">
This assignment has not been completed yet.
</p>

site: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw3/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw3/)

## Overview

TODO: TEST TEST TEST $a^2 + b^2 = c^2$

$$
x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$

## Part 1: Ray Generation and Scene Intersection
Let's utilize [Task 1](/hw3.md#task-1-generating-camera-rays) and [Task 2](/hw3.md#task-2-generation-pixel-samples) to discuss the ray generation and primitive intersection parts of the rendering pipeline. We will also explain primitive intersections and sphere intersections in [Task 4](/hw3.md#task-4-ray-sphere-intersection).

Then, we'll continue onto [Task 3](/hw3.md#task-3-ray-triangle-intersection) to explain the triangle intersection algorithm.

### Task 1: Generating Camera Rays
We first want to transform the image coordinates <code class="language-plaintext highlighter-rouge">(x, y)</code> to camera space by interpolating. Knowing that we're using an axis-aligned rectangular virtual camera sensor on the <code class="language-plaintext highlighter-rouge">Z = -1</code> plane, we use <code class="language-plaintext highlighter-rouge">hFov</code> and <code class="language-plaintext highlighter-rouge">vFov</code> field of view angles along the <code class="language-plaintext highlighter-rouge">X</code> and <code class="language-plaintext highlighter-rouge">Y</code> axis to transform into camera space.

Now, in 3-dimensional camera coordinations, we have the vector containing <code class="language-plaintext highlighter-rouge">x_camera</code>, <code class="language-plaintext highlighter-rouge">y_camera</code>, and <code class="language-plaintext highlighter-rouge">-1</code>, where the <code class="language-plaintext highlighter-rouge">*_camera</code> represents the respective point in camera space. From here, we transform the camera space ray to world space using the camera-to-world,<code class="language-plaintext highlighter-rouge">c2w_pos</code>, which is a 4x4 homogeneous coordinate system transform matrix given in lecture. We also make sure to normalize the ray's direction after the transform. Note specifically that since we placed the camera at <code class="language-plaintext highlighter-rouge">pos</code> (in world space), we utilize this as column 4.

Then, to set our range for the clipping planes, we utilized <code class="language-plaintext highlighter-rouge">nClip</code> and <code class="language-plaintext highlighter-rouge">fClip</code> as provided for <code class="language-plaintext highlighter-rouge">min_t</code> and <code class="language-plaintext highlighter-rouge">max_t</code> as directed.

### Task 2: Generation Pixel Samples
Then, now that we've generated our camera rays in world space, we now need to generate pixel samples!

We generate <code class="language-plaintext highlighter-rouge">ns_aa</code> random samples within the pixel, ensuring to normalize these coordinates. Then, we call <code class="language-plaintext highlighter-rouge">camera->generate_ray</code>, passing in these normalized <code class="language-plaintext highlighter-rouge">(x, y)</code> coordinates, and then estimate the radiance by calling <code class="language-plaintext highlighter-rouge">est_radiance_global_illumination</code>. Finally, once we've generated these <code class="language-plaintext highlighter-rouge">ns_aa</code> samples, we can average out the pixel color, and then update the <code class="language-plaintext highlighter-rouge">sampleBuffer</code> by calling <code class="language-plaintext highlighter-rouge">update_pixel</code> with that color.


### Task 3: Ray-Triangle Intersection
In order to implement our ray-triangle intersection algorithm, we had to modify the <code class="language-plaintext highlighter-rouge">Triangle::has_intersection</code> and <code class="language-plaintext highlighter-rouge">Triangle::intersect</code> methods. We created a helper function, <code class="language-plaintext highlighter-rouge">Triangle::test</code>, to match the code structure of the ray-sphere intersection, as well as to avoid rewriting code. Our overall strategy was to use the Möller-Trumbore intersection algorithm depicted in lecture.

Our <code class="language-plaintext highlighter-rouge">Triangle::has_intersection</code> method allows us to <code class="language-plaintext highlighter-rouge">test</code> whether a given ray <code class="language-plaintext highlighter-rouge">r</code> intersects a triangle and if so, updates the <code class="language-plaintext highlighter-rouge">t</code>, <code class="language-plaintext highlighter-rouge">u</code>, and <code class="language-plaintext highlighter-rouge">v</code> passed in.

To do so, we first check whether the ray and the plane the triangle lies on is parallel by determining whether the dot product between the two gives 0. If so, we return false (since a parallel ray and plane will never intersect). Then, we calculate <code class="language-plaintext highlighter-rouge">t</code>, <code class="language-plaintext highlighter-rouge">u</code>, and <code class="language-plaintext highlighter-rouge">v</code>. We calculate 
1. the cross product between the direction of the ray and $p_3 - p_1$
2. the cross product between the difference between the origin of the ray and $p_1$, and the difference between $p_2 - p_1$.

From here, we get 
- <code class="language-plaintext highlighter-rouge">t</code> is the dot product between (2) and $p_3 - p_1$,
- <code class="language-plaintext highlighter-rouge">u</code> is the dot product between (1) and the difference between the origin of the ray and $p_1$, and
- <code class="language-plaintext highlighter-rouge">v</code> is the division between the dot product of (2) and the direction of the ray by the dot product of (1) and $p_2 - p_1$.

Finally, we check to ensure that
- the intersection point is within the triangle
- <code class="language-plaintext highlighter-rouge">0</code> $\leq$ <code class="language-plaintext highlighter-rouge">min_t</code> $\leq$ <code class="language-plaintext highlighter-rouge">t</code> $\leq$ <code class="language-plaintext highlighter-rouge">max_t</code>

and then finally update <code class="language-plaintext highlighter-rouge">max_t</code> to complete our intersection.

Then, if an intersection occured, we populated <code class="language-plaintext highlighter-rouge">isect</code> as desired, filling in the surface normal, primitive, and bsdf.

After completing Task 3, our output for <code class="language-plaintext highlighter-rouge">./pathtracer -r 800 600 -f CBempty.png ../dae/sky/CBempty.dae</code> with normal shading is
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part1/task3_1.png" width="400px"/>
        <figcaption>sky/CBempty.dae with normal shading</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 4: Ray-Sphere Intersection
Like with [ray-triangle intersection](/hw3.md#task-3-ray-triangle-intersection), in order to implement our ray-triangle intersection algorithm, we had to modify the <code class="language-plaintext highlighter-rouge">Sphere::has_intersection</code> and <code class="language-plaintext highlighter-rouge">Sphere::intersect</code> methods (and inadvertently <code class="language-plaintext highlighter-rouge">Sphere::test</code>).

Like in our <code class="language-plaintext highlighter-rouge">Triangle</code> class, we use <code class="language-plaintext highlighter-rouge">Sphere::has_intersection</code> and <code class="language-plaintext highlighter-rouge">Sphere::intersect</code> as a way to call into our <code class="language-plaintext highlighter-rouge">Sphere::test</code>. 

Namely, within the <code class="language-plaintext highlighter-rouge">Sphere::test</code> function, we reduce the intersection points to the roots of a quadratic equation. We check whether the determinant is less than 0, noting that if so, this meant that the ray missed the sphere, and as such, we could immediately return false. We calculate the determinant by determining
1. the difference between the ray's origin and the origin of the sphere
2. the dot product between the ray's direction with itself
3. two times the dot product of the difference between the ray's origin and the origin of the sphere with the direction of the ray
4. and finally the dot product between the difference between the ray's origin and the origin of the sphere and itself subtracted by the radius of the sphere, squared

From here, we calculate the determinant by taking 
{% highlight js %}
(3)^2 - 4 * (2) * (4)
{% endhighlight %}

If we reached this step, we know that the ray and the sphere do intersect. Now, we take the square root of the determinant and select the closest <code class="language-plaintext highlighter-rouge">t</code> that still lies between <code class="language-plaintext highlighter-rouge">min_t</code> and <code class="language-plaintext highlighter-rouge">max_t</code>, using the quadratic formula to determine the candidate's time of intersection.

After calculating and ensuring that the minimally valid <code class="language-plaintext highlighter-rouge">t</code> candidate was selected (by ensuring that only intersections of <code class="language-plaintext highlighter-rouge">0</code> $\leq$ <code class="language-plaintext highlighter-rouge">min_t</code> $\leq$ <code class="language-plaintext highlighter-rouge">t</code> $\leq$ <code class="language-plaintext highlighter-rouge">max_t</code> are valid), we update <code class="language-plaintext highlighter-rouge">max_t</code> like desired to be the chosen candidate.

Then, if an intersection occured, we populated <code class="language-plaintext highlighter-rouge">isect</code> as desired, filling in the surface normal, primitive, and bsdf.

After completing Task 4, our output for <code class="language-plaintext highlighter-rouge">./pathtracer -r 800 600 -f CBspheres.png ../dae/sky/CBspheres_lambertian.dae</code> with normal shading is
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part1/task4_1.png" width="400px"/>
        <figcaption>sky/CBspheres_lambertian.dae with normal shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Part 2: Bounding Volume Hierarchy
In this part of the homework, we want to implement bounding volume hierarchy (BVH) such that we can speed up the rendering of our path tracer.

We will utilize [Task 1](/hw3.md#task-1-constructing-the-bvh), [Task 2](/hw3.md#task-2-intersecting-the-bounding-box), and [Task 3](/hw3.md#task-3-intersecting-the-bvh) to demonstrate our BVH construction algorithm. Within [Task 1](/hw3.md#task-1-constructing-the-bvh), we will explain the heuristic you choose for picking the splitting point.

### Task 1: Constructing the BVH
In <code class="language-plaintext highlighter-rouge">BVHAccel::construct_bvh</code>, we first generate the outermost bounding box. We iterate through the primitives passed in, and expand (union) that bounding box with our current one. 

If the number of primitives is less than or equal to the <code class="language-plaintext highlighter-rouge">max_leaf_size</code>, we create a new <code class="language-plaintext highlighter-rouge">BVHNode</code>, initializing its <code class="language-plaintext highlighter-rouge">start</code> and <code class="language-plaintext highlighter-rouge">end</code> to be the start and end of the primitives and return.

If not, we need to recurse. In doing so, we will need to first determine the split point and axis selection to split up our bounding volume hierarchy spatially. In reading [this paper](https://pbr-book.org/3ed-2018/Primitives_and_Intersection_Acceleration/Bounding_Volume_Hierarchies), we decided to first compute the average centroid across all the primitives. Then, we take the extent of the resulting <code class="language-plaintext highlighter-rouge">bbox</code>, which is a vector from opposing vertices of the box (creating a diagonal across the interior of the box). We determine which axis has the longest extent and use that as the axis to split on. 

Then, knowing our <code class="language-plaintext highlighter-rouge">avg_centroid</code>, we take the x, y, or z point given the axis that we chose to split on. Then, we call <code class="language-plaintext highlighter-rouge">std:stable_partition</code> on the primitives given in, checking whether that primitive's <code class="language-plaintext highlighter-rouge">centroid</code> value at the chosen axis, and checking whether that is less than the averaged centroid across the entire bounding box (for this level) at the chosen axis. If it was less than, we partitioned it in the first set and if greater than or equal two, in the second partition.

Finally, we checked that this start of the second partition was not equal to the original <code class="language-plaintext highlighter-rouge">start</code> or <code class="language-plaintext highlighter-rouge">end</code> (so one of the partitions wasn't empty), and if so, we assigned <code class="language-plaintext highlighter-rouge">node->l</code> and <code class="language-plaintext highlighter-rouge">node->r</code> to be recursive calls to <code class="language-plaintext highlighter-rouge">construct_bvh</code> given our two partitions so our <code class="language-plaintext highlighter-rouge">l</code> attribute was the first partition and the <code class="language-plaintext highlighter-rouge">r</code> attribute was the second partition.

We can see running <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -r 800 600 ../dae/meshedit/cow.dae</code> and then clicking the right arrow consecutively is

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part2/task1_1.png" width="100%"/>
        <figcaption>../dae/meshedit/cow.dae, base rendering</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw3/part2/task1_2.png" width="100%"/>
        <figcaption>../dae/meshedit/cow.dae, descending once to right child</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw3/part2/task1_3.png" width="100%"/>
        <figcaption>../dae/meshedit/cow.dae, descending twice to right child</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw3/part2/task1_4.png" width="100%"/>
        <figcaption>../dae/meshedit/cow.dae, descending thrice to right child</figcaption>
      </td>
    </tr>
  </table>
</div>

You can see that as we descend into right children in the bounding volume hierarchy, we are always splitting on the longest axis of the extent. For instance, for the first descent, we take the length of the cow, since that is the longest axis. Then, on the second descent, we take the height of the cow, since at that point, the height axis is longer than both the length and width axes. Finally, on the third descent, we split on the width axis of the cow, for the same reason.

### Task 2: Intersecting the Bounding Box
Now that we've constructed our bounding volume heirarchy, we want to check whether a ray intersects a giving bounding box. Implementing <code class="language-plaintext highlighter-rouge">BBox::intersect</code>, we utilize the given ray and axis-aligned plane intersection and ray and axis-aligned box intersection equations given in lecture.

Namely, since we want to represent time as $t = \frac{p_x' - o_x}{d_x}$, calculating perpendicular to x-axis, we calculate a <code class="language-plaintext highlighter-rouge">min_t</code> and a <code class="language-plaintext highlighter-rouge">max_t</code>. Then from here, we ensure that for all axes, <code class="language-plaintext highlighter-rouge">min_t[axis] < max_t[axis]</code>, swapping if that was not the case.

Finally, we calculate the interval of intersecting as the maximum of all the <code class="language-plaintext highlighter-rouge">min_t</code> axes and the minimum of all the <code class="language-plaintext highlighter-rouge">max_t</code> axes since we want to create the tighest bound. If the maximum <code class="language-plaintext highlighter-rouge">min_t</code> is greater than the minimum <code class="language-plaintext highlighter-rouge">max_t</code>, then we return false (there is no intersection since the ray missed the box), otherwise we set <code class="language-plaintext highlighter-rouge">t_0</code> and <code class="language-plaintext highlighter-rouge">t_1</code> accordingly and return true.

### Task 3: Intersecting the BVH
Finally, now that we have the ability to intersect a ray with a bounding box, we can now test that intersection with the BVH. Namely, here, we've completed <code class="language-plaintext highlighter-rouge">BVHAccel::has_intersection</code> and <code class="language-plaintext highlighter-rouge">BVHAccel:intersect</code>. We implement this using [this lecture slide](https://cs184.eecs.berkeley.edu/sp24/lecture/9-73/ray-tracing-and-acceleration-str) on BVH Recursive Traversal as a guide.

For <code class="language-plaintext highlighter-rouge">BVHAccel::has_intersection</code>, we first check whether the ray intersects the outermost bounding box and if not, we know it won't intersect with any primitive, so we can immediately return false. Then, we check that the intersection occurs such that <code class="language-plaintext highlighter-rouge">0</code> $\leq$ <code class="language-plaintext highlighter-rouge">min_t</code> $\leq$ <code class="language-plaintext highlighter-rouge">t</code> $\leq$ <code class="language-plaintext highlighter-rouge">max_t</code>, and if not, also return false. Finally, we perform a check if the current <code class="language-plaintext highlighter-rouge">node</code> is a leaf.
- If so, then we iterate through all the primitives of the <code class="language-plaintext highlighter-rouge">node</code>, incrementing the total <code class="language-plaintext highlighter-rouge">isects</code>, and then return true if for any of the primitives, it does intersect with the ray.
- Otherwise, we recursively check (by calling <code class="language-plaintext highlighter-rouge">BVHAccel::has_intersection</code> on the same <code class="language-plaintext highlighter-rouge">ray</code> and check whether it intersects with <code class="language-plaintext highlighter-rouge">node->l</code> or <code class="language-plaintext highlighter-rouge">node->r</code>).

<code class="language-plaintext highlighter-rouge">BVHAccel::intersect</code> functions similarly, except instead of the last step (when we return whether there is an intersection between the ray and <code class="language-plaintext highlighter-rouge">node->l</code> or <code class="language-plaintext highlighter-rouge">node->r</code>), we create two new <code class="language-plaintext highlighter-rouge">Intersection</code>s, calling <code class="language-plaintext highlighter-rouge">BVHAccel::intersect</code> recursively on <code class="language-plaintext highlighter-rouge">node->l</code> and <code class="language-plaintext highlighter-rouge">node->r</code>, passing in these two new <code class="language-plaintext highlighter-rouge">Intersection</code> objects. Then, we determine which is closer of the left hit or the right hit using boolean algebra and setting the parameter <code class="language-plaintext highlighter-rouge">Intersection</code> <code class="language-plaintext highlighter-rouge">i</code> to whichever of the hits was closer. Then, like previously, we return whether there was a recursive hit, bubbling up an intersection from the leaves.

### BVH Acceleration Analysis
Following the completion of [Part 2](/hw3.md#part-2-bounding-volume-hierarchy), here are how multiple different .dae files render. Without BVH acceleration, these files took much longer to render (as we'll discuss in the analysis below).
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part2/maxplanck.png" width="100%"/>
      <figcaption>../dae/meshedit/maxplanck.dae, rendered after BVH acceleration</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part2/cblucy.png" width="100%"/>
      <figcaption>../dae/sky/CBlucy.dae, rendered after BVH acceleration</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part2/dragon.png" width="100%"/>
      <figcaption>../dae/sky/dragon.dae, rendered after BVH acceleration</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part2/wall-e.png" width="100%"/>
      <figcaption>../dae/sky/wall-e.dae, rendered after BVH acceleration</figcaption>
    </td>
  </tr>
  </table>
</div>

The following data was collected by calling <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -r 800 600 -f {filename}.png ../dae/{path to file}.dae</code> with and without BVH acceleration.
<div align="center">
  <table class="with-internal-border">
    <colgroup>
      <col width="20%" />
      <col width="35%" />
      <col width="35%" />
    </colgroup>
    <thead>
      <tr class="header">
      <th>Filename</th>
      <th>Rendering Time without BVH Acceleration</th>
      <th>Rendering Time with BVH Acceleration</th>
    </tr>
    </thead>
    <tbody>
      <tr>
        <td markdown="span">../dae/meshedit/maxplanck.dae</td>
        <td markdown="span">47.1526s</td>
        <td markdown="span">0.0583s</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/dragon.dae</td>
        <td markdown="span">113.4021s</td>
        <td markdown="span">0.0462s</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/CBlucy.dae</td>
        <td markdown="span">168.5018s</td>
        <td markdown="span">0.0566s</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/wall-e.dae</td>
        <td markdown="span">414.7519s</td>
        <td markdown="span">0.0576s</td>
      </tr>
    </tbody>
  </table>
</div>

<div align="center">
  <table class="with-internal-border">
    <colgroup>
      <col width="20%" />
      <col width="35%" />
      <col width="35%" />
    </colgroup>
    <thead>
      <tr class="header">
      <th>Filename</th>
      <th>Rendering Avg Speed Per Second without BVH Acceleration</th>
      <th>Rendering Avg Speed Per Second BVH Acceleration</th>
    </tr>
    </thead>
    <tbody>
      <tr>
        <td markdown="span">../dae/meshedit/maxplanck.dae</td>
        <td markdown="span">0.0097 million rays/second</td>
        <td markdown="span">2.8513 million rays/second</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/dragon.dae</td>
        <td markdown="span">0.0031 million rays/second</td>
        <td markdown="span">3.6082 million rays/second</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/CBlucy.dae</td>
        <td markdown="span">0.0021 million rays/second</td>
        <td markdown="span">3.8665 million rays/second</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/wall-e.dae</td>
        <td markdown="span">0.0009 million rays/second</td>
        <td markdown="span">2.5576 million rays/second</td>
      </tr>
    </tbody>
  </table>
</div>

<div align="center">
  <table class="with-internal-border">
    <colgroup>
      <col width="20%" />
      <col width="35%" />
      <col width="35%" />
    </colgroup>
    <thead>
      <tr class="header">
      <th>Filename</th>
      <th>Avg Intersection Tests Per Ray without BVH Acceleration</th>
      <th>Avg Intersection Tests Per Ray with BVH Acceleration</th>
    </tr>
    </thead>
    <tbody>
      <tr>
        <td markdown="span">../dae/meshedit/maxplanck.dae</td>
        <td markdown="span">10501.9555582 tests/ray</td>
        <td markdown="span">6.901601 tests/ray</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/dragon.dae</td>
        <td markdown="span">24087.831808 tests/ray</td>
        <td markdown="span">4.790797 tests/ray</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/CBlucy.dae</td>
        <td markdown="span">34545.920619 tests/ray</td>
        <td markdown="span">3.804057 tests/ray</td>
      </tr>
      <tr>
        <td markdown="span">../dae/sky/wall-e.dae</td>
        <td markdown="span">62418.197373 tests/ray</td>
        <td markdown="span">7.574583 tests/ray</td>
      </tr>
    </tbody>
  </table>
</div>

We can see that from these data points, that the rendering speeds up from 800 to a couple thousand times from rendering without BVH acceleration to rendering with BVH acceleration. When we render based on the starter code, we know that we naively have to test intersection with the ray against all the primitives whereas when we use BVH acceleration, we test intersections with the ray against bounding boxes, which hierarchically, means that at the leaf level, we only test against a max of <code class="language-plaintext highlighter-rouge">max_leaf_size</code> primitives per leaf node that intersects with the ray. Furthermore, we see that this drastically increases the number of rays that we can test intersection with (since we no longer have to test for all primitives, as our bounding boxes split primitives in a tree-like structure). By enforcing that only the branches of the tree that intersect with the ray need to be traversed impacts the speed at which we can test rays, which greatly increases when we no longer have to test each ray with each primitive. Furthermore, we can see that we also decrease the total number of intersections per ray via the third table: this also makes sense because we no longer need to test a ray against all the thousands of primitives in the images, but rather a subset that will actually be intersected with by the hierarchy of the BVH. This mimics a logarithmic asymptotic, similar to data structures like binary trees. In sum, rendering with BVH acceleration extensively decreases the rendering time since we now need to perform less ray intersection tests with bounding boxes and primitives, which thus allows us to efficiently test through many more rays per second, speeding up the entire operation.

## Part 3: Direct Illumination

We will separately walk through the implementations of [uniform hemisphere sampling](/hw3.md#task-3-direct-lighting-with-uniform-hemisphere-sampling) and [importance sampling lights](/hw3.md#task-4-direct-lighting-by-importance-sampling-lights). First, though, we'll describe how to implement the [f and sample_f functions](/hw3.md#task-1-diffuse-bsdf) and [zero-bounce illumination](/hw3.md#task-2-zero-bounce-illumination).

### Task 1: Diffuse BSDF
In this task, we needed to implement <code class="language-plaintext highlighter-rouge">DiffuseBSDF::f</code> to return the evaluation of the BSDF, or reflectance in the given directions. We originally returned the <code class="language-plaintext highlighter-rouge">reflectance</code> of the <code class="language-plaintext highlighter-rouge">DiffuseBSDF</code>, but noticed this was too bright. Then, we divided this <code class="language-plaintext highlighter-rouge">reflectance</code> by $$2 * \pi$$ since the surface area of a unit hemisphere is $$2 * \pi$$, however this was too dark. We then chose to just divide <code class="language-plaintext highlighter-rouge">reflectance</code> by $$\pi$$ and this seemed to match the output as expected (we tested this while working on [Task 3](/hw3.md#task-3-direct-lighting-with-uniform-hemisphere-sampling)). Below we've attached the output of running <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -s 16 -l 8 -m 6 -H -r 480 360 ../dae/sky/CBbunny.dae</code>.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part3/task1_refl.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae,<br> using <code class="language-plaintext highlighter-rouge">f</code> = reflectance</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw3/part3/task1_2pi.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae,<br> using <code class="language-plaintext highlighter-rouge">f</code>= reflectance / (2 * PI)</figcaption>
      </td>
    </tr>
    <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw3/part3/task1_pi.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae,<br> using <code class="language-plaintext highlighter-rouge">f</code> = reflectance / PI</figcaption>
      </td>
    </tr>
  </table>
</div>

We see that using <code class="language-plaintext highlighter-rouge">reflectance</code> $$/ \pi$$ gave the brightness that best matched the desired output.

We also needed to implement <code class="language-plaintext highlighter-rouge">DiffuseBSDF::sample_f</code>, in which we sample a value based on the given <code class="language-plaintext highlighter-rouge">pdf</code> for <code class="language-plaintext highlighter-rouge">wi</code>, and then return <code class="language-plaintext highlighter-rouge">DiffuseBSDF::f</code> called with the passed in <code class="language-plaintext highlighter-rouge">wo</code> and our sampled <code class="language-plaintext highlighter-rouge">wi</code>.


### Task 2: Zero-bounce Illumination
To implement zero-bounce illumination, we took the given <code class="language-plaintext highlighter-rouge">Intersection</code> object, and accessed its <code class="language-plaintext highlighter-rouge">bsdf</code> attribute and called <code class="language-plaintext highlighter-rouge">get_emission()</code> to return the light that results from no bounces of light (the raw emission).

We then updated <code class="language-plaintext highlighter-rouge">est_radiance_global_illumination</code> to call <code class="language-plaintext highlighter-rouge">zero_bounce_radiance</code>, gaining the following output as expected for running <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -s 16 -l 8 -m 6 -H -f CBbunny_16_8.png -r 480 360 ../dae/sky/CBbunny.dae</code>.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part3/task2.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae, zero-bounce illumination</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 3: Direct Lighting with Uniform Hemisphere Sampling
Iterating through a total of <code class="language-plaintext highlighter-rouge">num_samples</code> samples, we first calculate the incoming light source value by sampling a <code class="language-plaintext highlighter-rouge">w_in</code> vector from the hemisphere and calculating its world-space coordinates. Then, we initialized a sample <code class="language-plaintext highlighter-rouge">Ray</code>, setting its <code class="language-plaintext highlighter-rouge">min_t</code> to <code class="language-plaintext highlighter-rouge">EPS_F</code>.

Then, creating an <code class="language-plaintext highlighter-rouge">Intersection</code>, we checked whether the bounding volume hierarchy intersected the sampled <code class="language-plaintext highlighter-rouge">Ray</code> by calling the <code class="language-plaintext highlighter-rouge">intersect</code> function from [Part 2 Task 3](/hw3.md#task-3-intersecting-the-bvh).

If there was an intersection from calling <code class="language-plaintext highlighter-rouge">intersect</code>, then we calculated the $$f_r$$, $$L_i$$, $$\cos_j$$, and $$pdf$$ to compute the reflection equation to get the outgoing lighting from this intersection.
- $$f_r$$: We call the function we wrote in [Task 1](/hw3.md#task-1-diffuse-bsdf) to calculate the BSDF.
- $$L_i$$: The incoming light is given by the emission of the intersection's BSDF.
- $$\cos_j$$: The dot product between the surface normal of the intersection and the world-space units of the ray gives the cosine angle between the two unit vectors.
- $$pdf$$: our pdf for hemisphere sampling is $$1 / (2 * \pi)$$ because the surface area for a unit sphere is $$4 \pi * r^2 = 4 \pi * 1^2 = 4 \pi$$, and hence, the surface area for a unit hemisphere is $$2 \pi$$, so the probability of sampling any point uniformly is $$1 \ (2 * \pi)$$.

Given these values, for each sample <code class="language-plaintext highlighter-rouge">Ray</code>, when an intersection existed, we added $$(f_r * L_i * \cos_j) / pdf$$ to our <code class="language-plaintext highlighter-rouge">L_out</code> value.

Finally, after performing <code class="language-plaintext highlighter-rouge">num_sample</code> samples, we normalized the outgoing light, <code class="language-plaintext highlighter-rouge">L_out</code>, by dividing by <code class="language-plaintext highlighter-rouge">num_samples</code>, returning this final <code class="language-plaintext highlighter-rouge">L_out</code>.

Running <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -s 64 -l 32 -m 6 -H -f {filename}.png -r 480 360 ../dae/sky/{filename}.dae</code> for uniform hemisphere sampling gave these two renders.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part3/task3_1.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw3/part3/task3_2.png" width="400px"/>
        <figcaption>../dae/sky/CBspheres_lambertian.dae</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 4: Direct Lighting by Importance Sampling Lights
For importance sampling, we performed a similar workflow as in [Task 3's hemisphere sampling](/hw3.md#task-3-direct-lighting-with-uniform-hemisphere-sampling) conceptually by calculating incoming light, and generating a sample ray to intersect with, but the structure of the looping was difference since we now were iterating through all the lights.

Particularly, iterating through all of <code class="language-plaintext highlighter-rouge">scene->lights</code>, we first checked whether <code class="language-plaintext highlighter-rouge">is_delta_light()</code> was true since if we were sampling a point light, it wouldn't matter what direction the ray was, the outgoing light would still be the same (so we would only need 1 sample, as opposed to <code class="language-plaintext highlighter-rouge">num_samples</code>, which we defined similarly to [Task 3](/hw3.md#task-3-direct-lighting-with-uniform-hemisphere-sampling)). 

Then, using however many samples we needed for that light as an index, we would iterate that many times, first calling the <code class="language-plaintext highlighter-rouge">light</code>'s <code class="language-plaintext highlighter-rouge">sample_L</code>, which returns the $$L_i$$ for this sample. Calling <code class="language-plaintext highlighter-rouge">sample_L</code> would populate
- <code class="language-plaintext highlighter-rouge">w_in</code> in world-space, so we would use the <code class="language-plaintext highlighter-rouge">w2o</code> matrix to also find the incoming light in local coordinates
- <code class="language-plaintext highlighter-rouge">distToLight</code> for the sample
- <code class="language-plaintext highlighter-rouge">pdf</code> that will be used to calculate outgoing light

We then utilized the resulting world-space incoming light vector and the <code class="language-plaintext highlighter-rouge">hit_p</code> to generate a <code class="language-plaintext highlighter-rouge">sample</code> <code class="language-plaintext highlighter-rouge">Ray</code>. We would set this <code class="language-plaintext highlighter-rouge">Ray</code>'s <code class="language-plaintext highlighter-rouge">min_t</code> to <code class="language-plaintext highlighter-rouge">EPS_F</code> and its <code class="language-plaintext highlighter-rouge">max_t</code> to <code class="language-plaintext highlighter-rouge">distToLight - EPS_F</code>.

Then, creating an <code class="language-plaintext highlighter-rouge">Intersection</code>, we checked whether the bounding volume hierarchy intersected the sampled <code class="language-plaintext highlighter-rouge">Ray</code> by calling the <code class="language-plaintext highlighter-rouge">intersect</code> function from [Part 2 Task 3](/hw3.md#task-3-intersecting-the-bvh).

If there wasn't an intersection from calling <code class="language-plaintext highlighter-rouge">intersect</code>, then we calculated the $$f_r$$ and $$pdf$$ (since the $$L_i$$ was returned from <code class="language-plaintext highlighter-rouge">sample_L</code> and the $$pdf$$ was populated from <code class="language-plaintext highlighter-rouge">sample_L</code>) to compute the reflection equation to get the outgoing lighting from this intersection. We note this difference from [Task 3](/hw3.md#task-3-direct-lighting-with-uniform-hemisphere-sampling) since we are only calculating if there wasn't an intersection since this means that the light source was not obstructed and thus, would actually hit this hit point. If there was an intersection, then we wouldn't actually want to illuminate this hit point because another object intersected with it first.
- $$f_r$$: We call the function we wrote in [Task 1](/hw3.md#task-1-diffuse-bsdf) to calculate the BSDF.
- $$\cos_j$$: The dot product between the surface normal of the intersection and the world-space units of the ray gives the cosine angle between the two unit vectors.

Given these values, for each sample <code class="language-plaintext highlighter-rouge">Ray</code>, when an intersection existed, we added $$(f_r * L_i * \cos_j) / pdf$$ to our <code class="language-plaintext highlighter-rouge">L_out</code> value.

Finally, after performing <code class="language-plaintext highlighter-rouge">num_sample</code> samples per light in <code class="language-plaintext highlighter-rouge">scene->lights</code> (or 1 if the light was an area light), we normalized the outgoing light, <code class="language-plaintext highlighter-rouge">L_out</code>, by dividing by <code class="language-plaintext highlighter-rouge">num_samples</code>, adding this outgoing light through this light to our desired outgoing light <code class="language-plaintext highlighter-rouge">L_out</code>. After aggregating across all the lights, we would return this final <code class="language-plaintext highlighter-rouge">L_out</code>.

While working on this task, we ran into an interesting debugging problem where the shadows seemingly rendered lighter than the areas where light should have hit, like shown in the bunny below.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part3/task4_shadow.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae with inverted shadows</figcaption>
      </td>
    </tr>
  </table>
</div>

We realized this was an error as we had kept our original code from [Task 3](/hw3.md#task-3-direct-lighting-with-uniform-hemisphere-sampling) to add to outgoing light if there was an intersection between the ray and the light--however, we soon realized we should only add to outgoing light if it were the opposite since this would mean there wouldn't be any obstruction from the light to the desired <code class="language-plaintext highlighter-rouge">hit_point</code>.

Running <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -s 64 -l 32 -m 6 -f {filename}.png -r 480 360 ../dae/sky/{filename}.dae</code> for importance sampling lights gave these three renders.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part3/task4_1.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw3/part3/task4_2.png" width="400px"/>
        <figcaption>../dae/sky/CBspheres_lambertian.dae</figcaption>
      </td>
    </tr>
    <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw3/part3/task4_3.png" width="400px"/>
        <figcaption>../dae/sky/dragon.dae</figcaption>
      </td>
    </tr>
  </table>
</div>

Now, using ../dae/sky/CBbunny.dae, we can compare the noise levels in soft shadows when rendering with 1, 4, 16, and 64 light rays, and with 1 sample per pixel by running the command <code class="language-plaintext highlighter-rouge">./pathtracer -t 8 -s 1 -l {num_rays} -m 6 -r 480 360 ../dae/sky/CBbunny.dae</code>.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part3/task4_l1s1.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 1 light ray, 1 sample per pixel</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part3/task4_l4s1.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 4 light rays, 1 sample per pixel</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part3/task4_l16s1.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 16 light rays, 1 sample per pixel</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part3/task4_l64s1.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 64 light rays, 1 sample per pixel</figcaption>
    </td>
  </tr>
  </table>
</div>

We can see that as the number of light rays increases, there's less noise in our outputted render. Particularly, the shadows of the bunny becomes more smoothed out, and the edges of shadows become softer (less harsh). The reasoning for this is because when there are less light rays, each shadow point calculated is clearer--each pixelated. However, when we add more light rays, there's more of a range (similar to when we supersampled in [Homework 1](/hw1.md#impacts-of-supersampling)). This allows for an overall smoother shadow where there are varying levels of grey that mark out the general portion of the shadow (darker) and the edges (lighter).

### Uniform Hemisphere Sampling v. Lighting Sampling
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part3/task3_1.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae<br>uniform hemisphere sampling</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part3/task4_1.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae<br>lighting sampling</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part3/task3_2.png" width="100%"/>
      <figcaption>../dae/sky/CBspheres_lambertian.dae<br>uniform hemisphere sampling</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part3/task4_2.png" width="100%"/>
      <figcaption>../dae/sky/CBspheres_lambertian.dae<br>lighting sampling</figcaption>
    </td>
  </tr>
  </table>
</div>

Overall, when we compare uniform hemisphere sampling to lighting sampling, we see that the lighting sampling outputs are much smoother and sharper as opposed to uniform hemisphere sampling, where there was a lot more graininess and noise. Notably, if we look at the walls, we see that the light that hits the walls isn't smoothly dispersed, but much more grainy in uniform hemisphere sampling. Since in uniform hemisphere sampling, we're taking samples in different directions around the given point, which means that in parts of an image that don't converge as quickly, they appear darker since they weren't sampled as much as needed, resulting in a grainy texture. In contrast, the lighting sampling we used placed importance of samples that actually impact the final lighting: we choose to include outgoing light only when it would be unobstructed and we sample directly from the light source rather than just points within the hemisphere itself. Therefore, since not all the samples of hemisphere sampling were not in the direction of the light source, there was a lot of noice as compared to lighting sampling, where we only get light and shadow rays since we only sample from lights, effectively removing all the noise.

## Part 4: Global Illumination
<!--
we like pretty spheres. https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/assets/hw3/part4/pretty_sphere.png
-->

### Task 1: Sampling with Diffuse BSDF
This task was a repeat of [Task 1](/hw3.md#task-1-diffuse-bsdf) from [Part 3](/hw3.md#part-3-direct-illumination).

### Task 2: Global Illumination with up to N Bounces of Light
Let's describe our implementation of the indirect lighting function. We first update <code class="language-plaintext highlighter-rouge">est_radiance_global_illumination</code> to first calculate <code class="language-plaintext highlighter-rouge">L_out</code> as the sum of the <code class="language-plaintext highlighter-rouge">zero_bounce_radiance</code> and <code class="language-plaintext highlighter-rouge">at_least_one_bounce_radiance</code>. We also modify <code class="language-plaintext highlighter-rouge">raytrace_pixel</code> to mark our camera ray's <code class="language-plaintext highlighter-rouge">depth</code> to be the <code class="language-plaintext highlighter-rouge">max_ray_depth</code>.

Within <code class="language-plaintext highlighter-rouge">at_least_one_bounce_radiance</code> is where the bulk of our remaining code resides. If the ray's depth is greater than 0, then we accumulate into <code class="language-plaintext highlighter-rouge">L_out</code> the return value of <code class="language-plaintext highlighter-rouge">one_bounce_radiance</code>. If the ray's depth is equal to 1, then this is the <code class="language-plaintext highlighter-rouge">L_out</code> that we actually return (if the ray's depth is less than 1, we return <code class="language-plaintext highlighter-rouge">Vector3D(0, 0, 0)</code>).

Then, if the ray's depth is greater than 1 (so we need to make a recursive call), we first take one random <code class="language-plaintext highlighter-rouge">sample_f</code> of the intersection's BSDF to get the incoming light <code class="language-plaintext highlighter-rouge">w_in</code> and calculate its world-space value.

Then, we generate a sample <code class="language-plaintext highlighter-rouge">Ray</code> from <code class="language-plaintext highlighter-rouge">hit_p</code> and the world-space <code class="language-plaintext highlighter-rouge">w_in</code>, setting the <code class="language-plaintext highlighter-rouge">Ray</code>'s <code class="language-plaintext highlighter-rouge">min_t</code> to <code class="language-plaintext highlighter-rouge">EPS_F</code> and <code class="language-plaintext highlighter-rouge">max_t</code> to <code class="language-plaintext highlighter-rouge">r.depth - 1</code>. This reflects the fact that we previously set the depth of the ray to the number of bounces specified on the command line, and here, our recursive call is decrementing the <code class="language-plaintext highlighter-rouge">depth</code> for each bounce we perform.

Then, we check whether nodes in our BVH intersect the sample ray, and if so, we calculate
- $$L_i$$: The recursive call to <code class="language-plaintext highlighter-rouge">at_least_one_bounce_radiance</code>, which will return to us the radiance from all later bounces.
- $$\cos_j$$: The dot product between the surface normal of the intersection and the world-space units of the ray gives the cosine angle between the two unit vectors.

Then, we accumulate into <code class="language-plaintext highlighter-rouge">L_out</code> the calculated $$(f_r * L_i * \cos_j) / pdf$$, returning <code class="language-plaintext highlighter-rouge">L_out</code>  at the end.

Using global illumination (now a combination of direct and indirect illumination), we render the following images using 1024 sampels per pixel:
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_global_illumination_spheres.png" width="100%"/>
      <figcaption>../dae/sky/CBspheres_lambertian.dae, global illumination</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_global_illumination_bunny.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, global illumination</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_global_illumination_blob.png" width="100%"/>
      <figcaption>../dae/sky/blob.dae, global illumination</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_global_illumination_dragon.png" width="100%"/>
      <figcaption>../dae/sky/dragon.dae, global illumination</figcaption>
    </td>
  </tr>
  </table>
</div>

### Direct v. Indirect Illumination
For both ../dae/sky/CBspheres_lambertian.dae and ../dae/sky/CBbunny.dae, we've rendered direct illumination (0 and 1 bounce) versus indirect illumination (> 1 bounce) at 1024 samples per pixel. Note in particular that the 0 and 1 bounce illumination resembles what we did in [Part 3](/hw3.md#part-3-direct-illumination) for zero- and one-bounce illumination while indirect illumination is what we accounted for in [Part 4](/hw3.md#part-4-global-illumination). Interestingly, we can see light now hitting the ceiling in the indirect illumination only renders as the light has come out of the area light, bounced off to some other surface, and then bounced to illuminate the ceiling.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_direct_illumination_spheres.png" width="100%"/>
      <figcaption>../dae/sky/CBspheres_lambertian.dae, direct illumination (0 and 1 bounce)</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_indirect_illumination_spheres.png" width="100%"/>
      <figcaption>../dae/sky/CBspheres_lambertian.dae, indirect illumination (> 1 bounce)</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_direct_illumination_bunny.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, direct illumination (0 and 1 bounce)</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_indirect_illumination_bunny.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, indirect illumination (> 1 bounce)</figcaption>
    </td>
  </tr>
  </table>
</div>

### Adding Non-Accumulation Rendering
In order to account for rendering non-accumulated bounces, we needed to utilize the <code class="language-plaintext highlighter-rouge">isAccumBounces</code> variable of the <code class="language-plaintext highlighter-rouge">PathTracer</code> class. Namely, we know that when <code class="language-plaintext highlighter-rouge">isAccumBounces</code> is false, we only want to add the outgoing light for the last bounce, and for all other bounces, return <code class="language-plaintext highlighter-rouge">Vector3D(0, 0, 0)</code>.

To facilitate this, we updated two things, namely just instances where we add to <code class="language-plaintext highlighter-rouge">L_out</code>.
- Instead of always adding <code class="language-plaintext highlighter-rouge">one_bounce_radiance</code> to <code class="language-plaintext highlighter-rouge">L_out</code>, we only call this when <code class="language-plaintext highlighter-rouge">isAccumBounces</code> is true or the depth of the ray is one (which means we've hit the last bounce).
- Instead of accumulating to <code class="language-plaintext highlighter-rouge">L_out</code> (summing), we directly set <code class="language-plaintext highlighter-rouge">L_out</code> equal to $$(f_r * L_i * \cos_j) / pdf$$ if <code class="language-plaintext highlighter-rouge">isAccumBounces</code> is false since we don't want to accumulate the outgoing light, but instead, just return $$(f_r * L_i * \cos_j) / pdf$$, noting that $$L_i$$ is the output of our recursive call to <code class="language-plaintext highlighter-rouge">at_least_one_bounce_radiance</code>.

Here, in two columns, we shown the output of ../dae/sky/CBbunny.dae sampled 1024 times per pixel, from the 0th to 5th bounce of light, where the left column accumulates the light (so <code class="language-plaintext highlighter-rouge">isAccumBouces</code> is true) while the right column does not accumulate the outgoing light and instead just shows the light from that specific bounce (<code class="language-plaintext highlighter-rouge">isAccumBounces</code> is false).

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m0accum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 0th bounce of light, with accumulation</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m0noaccum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 0th bounce of light, no accumulation</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m1accum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 1st bounce of light, with accumulation</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m1noaccum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 1st bounce of light, no accumulation</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m2accum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 2nd bounce of light, with accumulation</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m2noaccum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 2nd bounce of light, no accumulation</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m3accum.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, 3rd bounce of light, with accumulation</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m3noaccum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 3rd bounce of light, no accumulation</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m4accum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 4th bounce of light, with accumulation</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m4noaccum.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, 4th bounce of light, no accumulation</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m5accum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 5th bounce of light, with accumulation</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_m5noaccum.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 5th bounce of light, no accumulation</figcaption>
    </td>
  </tr>
  </table>
</div>
Between the second and third bounce of light, we see that the image gets a lot darker overall (the bunny texture isn't as clear around its legs), however, it looks like the edges of the floor still retain some light. Overall, we can understand why because after the second bounce, it makes sense that the light has already dissapated. The decrease in intensity of outgoing light experienced here could also be due to the fact that the light could have also been absorbed by the bunny or the walls. In the second bounce, light that hit the ground would've hit the bottom side of the bunny, explaining why its legs were most likely illuminated. By the third bounce, the only really light areas with higher density of sharpened light seems to be near the edges of the floor in the back and a little halo under the bunny.

In contrast to purely rasterized images, we know that adding bounces and global illumination definitely impacts the realism of the image since we're now able to add complex shadows and diffuse shadows and colors more smoothly across images. Pure rasterization did not allow for the interaction of light within a scene, which meant that prior to our work here, in previous projects, we were unable to render anything out of a controlled setting--it was always just one item in empty space. Now, we have the ability to utilize lights in our staging and mimic shadows and perspective similarly how we perceive three-dimensional objects in real life. Pretty neat!

### Pixel Sampling Rate Variation

Next, we render ../dae/sky/CBbunny.dae at 4 light rays with various sample-per-pixel rates.
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_bunny_s1l4.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 1 sample per pixel, 4 light rays</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_bunny_s2l4.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 2 samples per pixel, 4 light rays</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_bunny_s4l4.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 4 samples per pixel, 4 light rays</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_bunny_s8l4.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 8 samples per pixel, 4 light rays</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/task2_bunny_s16l4.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 16 samples per pixel, 4 light rays</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/task2_bunny_s64l4.png" width="100%"/>
      <figcaption>../dae/sky/CBbunny.dae, 64 samples per pixel, 4 light rays</figcaption>
    </td>
  </tr>
  <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw3/part4/task2_bunny_s1024l4.png" width="400px"/>
        <figcaption>../dae/sky/CBbunny.dae, 1024 samples per pixel, 4 light rays</figcaption>
      </td>
    </tr>
  </table>
</div>

We see that as the pixel sampling rate increases, the bunny gets smoother, clearer, and less grainier: the perks of higher sampling rates!

### Task 3: Global Illumination with Russian Roulette
TODO: explain additions for russian roulette

Here we render ../dae/sky/CBbunny.dae with Russian Roulette with 1024 samples per pixel, setting the maximum ray depth set to 0, 1, 2, 3, 4, and 100.
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/russian_bunny_0.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, Russian Roulette rendering up to 0 0 bounce</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/russian_bunny_1.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, Russian Roulette rendering up to 1 bounce</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/russian_bunny_2.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, Russian Roulette rendering up to 2 bounces</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/russian_bunny_3.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, Russian Roulette rendering up to 3 bounces</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw3/part4/russian_bunny_4.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, Russian Roulette rendering up to 4 bounces</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw3/part4/russian_bunny_100.png" width="100%"/>
      <figcaption>.../dae/sky/CBbunny.dae, Russian Roulette rendering up to 100 bounces</figcaption>
    </td>
  </tr>
  </table>
</div>

## Part 5: Adaptive Sampling
TODO

## Contributors
Edward Park, Ashley Chiu

TODO: At the end, if you worked with a partner, please write a short paragraph together for your final report that describes how you collaborated, how it went, and what you learned.