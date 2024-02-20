---
layout: page
title: 'Homework 2: MeshEdit'
has_right_toc: true
---
<p class="warning-message">
This assignment has not been completed yet.
</p>

site: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw2/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw2/)

## Overview

## Task 1: Bezier curves with 1D de Casteljau subdivision

The de Casteljau algorithm takes in a set of control points and recursively finds intermediate points until its stopping condition (when there remains 1 point), when we hit this case, this one point will exist on the Bezier curve generated from the original control points.

In depth, de Casteljau takes in <code class="language-plaintext highlighter-rouge">n</code> initial control point and a parameter <code class="language-plaintext highlighter-rouge">t</code> between 0 and 1 (shifting <code class="language-plaintext highlighter-rouge">t</code> ultimately gives you all the points that lie on the Bezier curve). The algorithm in return, in one step, returns <code class="language-plaintext highlighter-rouge">n - 1</code> intermediate points, and recursively applying the algorithm gets us from <code class="language-plaintext highlighter-rouge">n</code> points down to <code class="language-plaintext highlighter-rouge">1</code> in <code class="language-plaintext highlighter-rouge">n - 1</code> steps.

Here, in <code class="language-plaintext highlighter-rouge">BezierCurve::evaluateStep</code>, we have implemented the method to get from <code class="language-plaintext highlighter-rouge">n</code> points to <code class="language-plaintext highlighter-rouge">n - 1</code> intermediary points, effectively executing one step of de Casteljau's. At each step, given the points <code class="language-plaintext highlighter-rouge">p_1</code> to <code class="language-plaintext highlighter-rouge">p_k</code>, for each <code class="language-plaintext highlighter-rouge">p_i'</code>, we linearly interpolate <code class="language-plaintext highlighter-rouge">(1 - t) * p_i + t * p_{i + 1}</code>. Therefore, to generate the Bezier curve, we call <code class="language-plaintext highlighter-rouge">evaluateStep</code> recursively, <code class="language-plaintext highlighter-rouge">n - 1</code> times, to get our final point that lies on the Bezier curve.

Below, we show screenshots of each step of the evaluation from the original control points down to the final evaluated point.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task1_1.png" width="100%"/>
        <figcaption>Original control points</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1_2.png" width="100%"/>
        <figcaption>Step 1</figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw2/task1_3.png" width="100%"/>
        <figcaption>Step 2</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1_4.png" width="100%"/>
        <figcaption>Step 3</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task1_5.png" width="100%"/>
        <figcaption>Step 4</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1_6.png" width="100%"/>
        <figcaption>Step 5</figcaption>
      </td>
    </tr>
  </table>
</div>

Here's the completed Bezier curve constructed from running de Casteljau's algorithm visually above.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task1_7.png" width="400px"/>
        <figcaption>Completed Bezier curve</figcaption>
      </td>
    </tr>
  </table>
</div>

We can also drag and move the original control points that we've shown above to show a slightly different Bezier curve. Here, we show the curve

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task1_8.png" width="400px"/>
        <figcaption>Moving original points to create a slightly different Bezier curve</figcaption>
      </td>
    </tr>
  </table>
</div>

and here we show how we can modify the parameter <code class="language-plaintext highlighter-rouge">t</code> as this shows us other points that will lie along the green Bezier curve.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task1_9.png" width="100%"/>
        <figcaption>Decreasing <code class="language-plaintext highlighter-rouge">t</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1_10.png" width="100%"/>
        <figcaption>Increasing <code class="language-plaintext highlighter-rouge">t</code></figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 2: Bezier surfaces with separable 1D de Casteljau

To extend de Casteljau's algorithm to Bezier surfaces, conceptually, instead of utilizing a vector of points, we analyze a matrix. Namely, given an <code class="language-plaintext highlighter-rouge">n x n</code> matrix of control points, we first perform the 1D de Casteljau algorithm on each of the rows to get one point per row (for each <code class="language-plaintext highlighter-rouge">u</code>), and then perform the algorithm and interpolation again on each of those points to get the final point (in the <code class="language-plaintext highlighter-rouge">v</code> direction). This results in one point that lies on the Bezier surface at the given <code class="language-plaintext highlighter-rouge">u</code> and <code class="language-plaintext highlighter-rouge">v</code>.

To implement this, we modified three functions: <code class="language-plaintext highlighter-rouge">BezierPatch::evaluateStep</code>, <code class="language-plaintext highlighter-rouge">BezierPatch::evaluate1D</code>, and <code class="language-plaintext highlighter-rouge">BezierPatch::evaluate</code>.

### <code class="language-plaintext highlighter-rouge">BezierPatch::evaluateStep</code>
An extension of our previous  <code class="language-plaintext highlighter-rouge">BezierCurve::evaluateStep</code>, the code is essentially teh same except we now have  <code class="language-plaintext highlighter-rouge">Vector3D</code>s instead of  <code class="language-plaintext highlighter-rouge">Vector2D</code>s.

### <code class="language-plaintext highlighter-rouge">BezierPatch::evaluate1D</code>
Calling <code class="language-plaintext highlighter-rouge">BezierPatch::evaluateStep</code> <code class="language-plaintext highlighter-rouge">n - 1</code> times, the purpose of this function is to fully evaluate de Casteljau's algorithm for one set of points. This takes in <code class="language-plaintext highlighter-rouge">n</code> points and actually only outputs 1 (in comparison to our two <code class="language-plaintext highlighter-rouge">evaluateStep</code> functions that output <code class="language-plaintext highlighter-rouge">n - 1</code> points).

### <code class="language-plaintext highlighter-rouge">BezierPatch::evaluate</code>
Our final method in the sequence, this takes the effort to combine our previous methods and evaluate de Casteljau's algorithm on each of the <code class="language-plaintext highlighter-rouge">n</code> rows, storing each of the points in a vector, and then calling <code class="language-plaintext highlighter-rouge">evaluate1D</code> on the resulting vector. Importantly, given that we have parameters <code class="language-plaintext highlighter-rouge">u</code> and <code class="language-plaintext highlighter-rouge">v</code> that resemble our parameter <code class="language-plaintext highlighter-rouge">t</code> from [Task 1](/hw2.md#task-1-bezier-curves-with-1d-de-casteljau-subdivision), for our looping mechanism to <code class="language-plaintext highlighter-rouge">evaluate1D</code> on all the rows, we pass in <code class="language-plaintext highlighter-rouge">u</code> so the points we're returned for each of the rows is at the same place within each row. Then, calling <code class="language-plaintext highlighter-rouge">evaluate1D</code> to combine all the points returned to us there, we pass in <code class="language-plaintext highlighter-rouge">v</code>, to get the correct column location that we want to us. This results in a point on the Bezier surface. We can think that if we iterate over all <code class="language-plaintext highlighter-rouge">u</code> and <code class="language-plaintext highlighter-rouge">v</code> values, we would map out every point on the Bezier surface using de Casteljau's.

In conjunction, the three methods abstract away what the previous did: evaluating in one dimension depended on evaluating at one particular step while fully evaluating using de Casteljau had to abstract away evaluating a set of points in one dimension. Below is a screenshot of bez/teapot.bez, rendered following our implementation of de Casteljau's algorithm for Bezier surfaces.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task2.png" width="400px"/>
        <figcaption>bez/teapot.bez evaluated by de Casteljau</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 3: Area-weighted vertex normals

To implement area-weighted vertex normals, we first notice that we are working within the <code class="language-plaintext highlighter-rouge">Vertex</code> class (since the method is <code class="language-plaintext highlighter-rouge">Vertex::normal</code>), which means we can get the halfedge for this vertex to start. Then, we loop through all the vertices surrounding this vertex (stopping when we reach the beginning again) and catch the other two points in the triangle (<code class="language-plaintext highlighter-rouge">curr->next()->vertex()</code> and <code class="language-plaintext highlighter-rouge">curr->next()->next()->vertex()</code>). Marking their positions, we get the vectors of the two edges from the <code class="language-plaintext highlighter-rouge">Vertex</code> instance we're currently in to those two positions and then take the cross product of the two edge vectors. 

Namely, because the cross product of two edges of a triangle is area-weighted normal vector of the triangle, we want to generate a sum of these to then take the normal of. We iterate through, now, working with the next of the twin of the current halfedge, until we reach back to the starting halfedge again.

Doing this traversal guarantees that we have traversed all the faces that are incident to this vertex (since we are moving to the direct adjacent face through the <code class="language-plaintext highlighter-rouge">twin()</code> traversal), until we reach back to the start.

Now, since we've retrieved a sum of all the normal vectors of each of the triangles, we normalize this summed vector to return. We note that this resulting normalized vector is the area-weighted normal at the given vertex, as desired.

We can see in the image below that the shading becomes a lot smoother, and we've implemented Phong shading!

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task3_1.png" width="100%"/>
        <figcaption>dae/teapot.dae with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task3_2.png" width="100%"/>
        <figcaption>dae/teapot.dae with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 4: Edge flip

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4_1.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task4_2.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge flips with Phong shading</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4_3.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task4_4.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge flips with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 5: Edge split

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4_1.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge splits with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task4_2.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge splits with Phong shading</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5_1.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5_2.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits with Phong shading</figcaption>
      </td>
    </tr>
     <tr>
      <td align="center">
        <img src="../assets/hw2/task5_3.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits and flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5_4.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits and flips with Phong shading</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5_5.png" width="100%"/>
        <figcaption>dae/teapot.dae, even more edge splits and flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5_6.png" width="100%"/>
        <figcaption>dae/teapot.dae, even more edge splits and flips with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

### Extra Credit (beep beep!)
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5_extracredit1.png" width="400px"/>
        <figcaption>dae/beetle.dae, highlighting a boundary edge</figcaption>
      </td>
    </tr>
  </table>
</div>

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5_extracredit2.png" width="100%"/>
        <figcaption>dae/beetle.dae, splitting a boundary edge with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5_extracredit3.png" width="100%"/>
        <figcaption>dae/beetle.dae, splitting a boundary edge with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 6: Loop subdivision for mesh upsampling

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_1.png" width="100%"/>
        <figcaption>dae/cube.dae, loaded</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_2.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 1</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_3.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 2</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_4.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 3</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_5.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 4</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_6.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5</figcaption>
      </td>
    </tr>
  </table>
</div>

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_7.png" width="100%"/>
        <figcaption>dae/cube.dae, loaded, pre-processed with one split on each diagonal edge on each face</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_8.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 1, after pre-processing</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_9.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 2, after pre-processing</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_10.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 3, after pre-processing</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_11.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 4, after pre-processing</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_12.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5, after pre-processing</figcaption>
      </td>
    </tr>
  </table>
</div>

comparing with no pre-processing
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6_6.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5, no pre-processing</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6_12.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5, after pre-processing</figcaption>
      </td>
    </tr>
  </table>
</div>

## Contributors
Ashley Chiu, Angel Aldaco
