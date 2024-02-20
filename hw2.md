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

Here, in <code class="language-plaintext highlighter-rouge">BezierCurve::evaluateStep</code>, we have implemented the method to get from <code class="language-plaintext highlighter-rouge">n</code> points to <code class="language-plaintext highlighter-rouge">n - 1</code> intermediary points, effectively executing one step of de Casteljau's. At each step, given the points <code class="language-plaintext highlighter-rouge">p1</code> to <code class="language-plaintext highlighter-rouge">pk</code>, for each <code class="language-plaintext highlighter-rouge">pi'</code>, we linearly interpolate <code class="language-plaintext highlighter-rouge">(1 - t) * p_i + t * p_{i + 1}</code>.

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
