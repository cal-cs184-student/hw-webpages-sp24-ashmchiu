---
layout: page
title: 'Homework 3: Pathtracer'
has_right_toc: true
---

<p class="warning-message">
This assignment has not been completed yet.
</p>

site: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw3/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw3/)

## Overview

## Part 1: Ray Generation and Scene Intersection
Let's utilize [Task 1](/hw3.md#task-1-generating-camera-rays) and [Task 2](/hw3.md#task-2-generation-pixel-samples) to discuss the ray generation and primitive intersection parts of the rendering pipeline. We will also explain primitive intersections and sphere intersections in [Task 4](/hw3.md#task-4-ray-sphere-intersection).

Then, we'll continue onto [Task 3](/hw3.md#task-3-ray-triangle-intersection) to explain the triangle intersection algorithm.

### Task 1: Generating Camera Rays
We first want to transform the image coordinates <code class="language-plaintext highlighter-rouge">(x, y)</code> to camera space by interpolating. Knowing that we're using an axis-aligned rectangular virtual camera sensor on the <code class="language-plaintext highlighter-rouge">Z = -1</code> plane, given our <code class="language-plaintext highlighter-rouge">hFov</code> and <code class="language-plaintext highlighter-rouge">vFov</code> field of view angles along the <code class="language-plaintext highlighter-rouge">X</code> and <code class="language-plaintext highlighter-rouge">Y</code> axis.

Now, in 3-dimensional camera coordinations, we have the vector containing <code class="language-plaintext highlighter-rouge">x_camera</code>, <code class="language-plaintext highlighter-rouge">y_camera</code>, and <code class="language-plaintext highlighter-rouge">1</code>, where the <code class="language-plaintext highlighter-rouge">*_camera</code> represents the respective point in camera space. From here, we transform the camera space ray to world space using the camera-to-world,<code class="language-plaintext highlighter-rouge">c2w_pos</code>, matrix given in lecture, normalizing it in the process. Note specifically that since we placed the camera at <code class="language-plaintext highlighter-rouge">pos</code> (in world space), we utilize this as column 4.

Then, to set our range for the clipping planes, we utilized <code class="language-plaintext highlighter-rouge">nClip</code> and <code class="language-plaintext highlighter-rouge">fClip</code> as provided for <code class="language-plaintext highlighter-rouge">min_t</code> and <code class="language-plaintext highlighter-rouge">max_t</code> as directed.

### Task 2: Generation Pixel Samples
Then, now that we've generated our camera rays in world space, we now need to generate pixel samples!

We generate <code class="language-plaintext highlighter-rouge">ns_aa</code> random samples, ensuring to normalize these coordinates. Then, we call <code class="language-plaintext highlighter-rouge">camera->generate_ray</code>, passing in these normalized <code class="language-plaintext highlighter-rouge">(x, y)</code> coordinates, and then estimate the radiance by calling <code class="language-plaintext highlighter-rouge">est_radiance_global_illumination</code>. Finally, once we've generated these <code class="language-plaintext highlighter-rouge">ns_aa</code> samples, we can average out the pixel color, and then call <code class="language-plaintext highlighter-rouge">update_pixel</code> with that color in our <code class="language-plaintext highlighter-rouge">sampleBuffer</code>.


### Task 3: Ray-Triangle Intersection
In order to implement our ray-triangle intersection algorithm, we had to modify the <code class="language-plaintext highlighter-rouge">Triangle::has_intersection</code> and <code class="language-plaintext highlighter-rouge">Triangle::intersect</code> methods (and inadvertently, <code class="language-plaintext highlighter-rouge">Triangle::test</code>). Our overall strategy was to use the Möller-Trumbore intersection algorithm depicted in lecture.

Our <code class="language-plaintext highlighter-rouge">Triangle::has_intersection</code> method allows us to <code class="language-plaintext highlighter-rouge">test</code> whether a given ray <code class="language-plaintext highlighter-rouge">r</code> intersects a triangle and if so, updates the <code class="language-plaintext highlighter-rouge">t</code>, <code class="language-plaintext highlighter-rouge">u</code>, and <code class="language-plaintext highlighter-rouge">v</code> passed in.

To do so, we first check whether the ray and the plane the triangle lies on is parallel by determining whether the dot product between the two gives 0. If so, we return false (since a parallel ray and plane will never intersect). Then, we calculate <code class="language-plaintext highlighter-rouge">t</code>, <code class="language-plaintext highlighter-rouge">u</code>, and <code class="language-plaintext highlighter-rouge">v</code>. We calculate 
1. the cross product between the direction of the ray and <code class="language-plaintext highlighter-rouge">p_3 - p_1</code>
2. the cross product between the difference between the origin of the ray and the difference between <code class="language-plaintext highlighter-rouge">p_2 - p_1</code>

From here, we get 
- <code class="language-plaintext highlighter-rouge">t</code> is the dot product between (2) and <code class="language-plaintext highlighter-rouge">p_3 - p_1</code>,
- <code class="language-plaintext highlighter-rouge">u</code> is the dot product between (1) and the difference between the origin of the ray and <code class="language-plaintext highlighter-rouge">p_1</code>, and
- <code class="language-plaintext highlighter-rouge">v</code> is the division between the dot product of (2) and the direction of the ray by the dot product of (1) and <code class="language-plaintext highlighter-rouge">p_2 - p_1</code>.

Finally, we check to ensure that
- the intersection point is within the triangle
- <code class="language-plaintext highlighter-rouge">0 <= min_t <= t <= max_t</code>

and then finally update <code class="language-plaintext highlighter-rouge">max_t</code> to complete our intersection.

Then, if an intersection occured, we populated <code class="language-plaintext highlighter-rouge">isect</code> as desired, filling in the surface normal, primitive, and bsdf.

After completing Task 3, our output for <code class="language-plaintext highlighter-rouge">./pathtracer -r 800 600 -f CBempty.png ../dae/sky/CBempty.dae</code> with normal shading is
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part1/part1_task3_1.png" width="400px"/>
        <figcaption>sky/CBempty.dae with normal shading</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 4: Ray-Sphere Intersection
Like with [ray-triangle intersection](/hw3.md#task-3-ray-triangle-intersection), in order to implement our ray-triangle intersection algorithm, we had to modify the <code class="language-plaintext highlighter-rouge">Sphere::has_intersection</code> and <code class="language-plaintext highlighter-rouge">Sphere::intersect</code> methods (and inadvertently <code class="language-plaintext highlighter-rouge">Sphere::test</code>).

Like in our <code class="language-plaintext highlighter-rouge">Triangle</code> class, we use <code class="language-plaintext highlighter-rouge">Sphere::has_intersection</code> and <code class="language-plaintext highlighter-rouge">Sphere::intersect</code> as a way to call into our <code class="language-plaintext highlighter-rouge">Sphere::test</code>. 

Namely, within the <code class="language-plaintext highlighter-rouge">Sphere::test</code> function, we check whether the determinant is less than 0, noting that if so, this meant that the ray missed the sphere, and as such, we could immediately return false. We calculate the determinant by determining
1. the difference between the ray's origin and the origin of the sphere
2. the dot product between the ray's direction with itself
3. two times the dot product of the difference between the ray's origin and the origin of the sphere with the direction of the ray
4. and finally the dot product between the difference between the ray's origin and the origin of the sphere and itself subtracted by the radius of the sphere, squared

From here, we calculate the determinant by taking 
{% highlight js %}
(3)^2 - 4 * (2) * (4)
{% endhighlight %}

If we reached this step, we know that the ray and the sphere do intersect. Now, we take the square root of the determinant and select the closest <code class="language-plaintext highlighter-rouge">t</code> that still lies between <code class="language-plaintext highlighter-rouge">min_t</code> and <code class="language-plaintext highlighter-rouge">max_t</code>, using the quadratic formula to determine the candidate's time of intersection.

After calculating and ensuring that the minimally valid <code class="language-plaintext highlighter-rouge">t</code> candidate was selected (by ensuring that only intersections of <code class="language-plaintext highlighter-rouge">0 <= min_t <= t <= max_t</code> are valid), we update <code class="language-plaintext highlighter-rouge">max_t</code> like desired to be the chosen candidate.

Then, if an intersection occured, we populated <code class="language-plaintext highlighter-rouge">isect</code> as desired, filling in the surface normal, primitive, and bsdf.

After completing Task 4, our output for <code class="language-plaintext highlighter-rouge">./pathtracer -r 800 600 -f CBspheres.png ../dae/sky/CBspheres_lambertian.dae</code> with normal shading is
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw3/part1/part1_task4_1.png" width="400px"/>
        <figcaption>sky/CBspheres_lambertian.dae with normal shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Part 2: Bounding Volume Hierarchy

### Task 0: Timing Experiment

### Task 1: Constructing the BVH

### Task 2: Intersecting the Bounding Box

### Task 3: Intersecting the BVH

## Part 3: Direct Illumination

### Task 1: Diffuse BSDF

### Task 2: Zero-bounce Illumination

### Task 3: Direct Lighting with Uniform Hemisphere Sampling

### Task 4: Direct Lighting by Importance Sampling Lights

## Part 4: Global Illumination

### Task 1: Sampling with Diffuse BSDF

### Task 2: Global Illumination with up to N Bounces of Light

### Task 3: Global Illumination with Russian Roulette

## Part 5: Adaptive Sampling

## Contributors
Edward Park, Ashley Chiu