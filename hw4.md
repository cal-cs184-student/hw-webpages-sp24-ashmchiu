---
layout: page
title: 'Homework 4: Clothsim'
has_right_toc: true
---
<p class="warning-message">
This assignment has not been completed yet.
</p>

## Overview
Todo

## Part 1: Masses and springs
In this part, our main goal was creating a grid of point masses and springs. To do so, we iterated through <code class="language-plaintext highlighter-rouge">num_height_points</code> and an inner loop of <code class="language-plaintext highlighter-rouge">num_width_points</code> to generate our point masses in row major order. Depending on whether the orientation was horizontal or vertical, we either varied across the <code class="language-plaintext highlighter-rouge">xz</code> plane or the <code class="language-plaintext highlighter-rouge">xy</code> plane. Furthermore, if the point mass's <code class="language-plaintext highlighter-rouge">(x, y)</code> index was within the cloth's <code class="language-plaintext highlighter-rouge">pinned</code> vector, then we set their <code class="language-plaintext highlighter-rouge">pinned</code> boolean to <code class="language-plaintext highlighter-rouge">true</code> (which we'll see at the corners of ../scene/pinned4.json).

Now that we've created our grid of point masses, we then created our springs with structural, shearing, and bending constraints. We note that
- structural constraints were applied between a point mass and the point masses to its left and directly above it (if they existed)
- shearing constraints were applied between a point mass and the point masses to its diagonal upper left and right (if they existed)
- bending constraints were applied between a point mass and the point masses two to the left and two above it (if they existed it).

Below, we've included several screenshots of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/pinned2.json</code> from viewing angles to show the wireframe and structure of point masses and springs.
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part1/pinned.png" width="100%"/>
      <figcaption>../scene/pinned2.json, entire grid in frame</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part1/pinned_zoom.png" width="100%"/>
      <figcaption>../scene/pinned2.json, straight zoom in</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part1/all_tilted.png" width="100%"/>
      <figcaption>../scene/pinned2.json, titled view of grid</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part1/all_tilted_zoom.png" width="100%"/>
      <figcaption>../scene/pinned2.json, titled view zoom in</figcaption>
    </td>
  </tr>
  </table>
</div>

Now, here are some screenshots of ../scene/pinned2.json without any shearing constraints, with only shearing constraints, and with all constraints.
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part1/no_shear.png" width="100%"/>
      <figcaption>../scene/pinned2.json, without any shearing constraints</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part1/shear.png" width="100%"/>
      <figcaption>../scene/pinned2.json, with only shearing constraints</figcaption>
    </td>
  </tr>
  <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw4/part1/all.png" width="400px"/>
        <figcaption>../scene/pinned2.json, with all constraints</figcaption>
    </td>
    </tr>
  </table>
</div>


## Part 2: Simulation via numerical integration

## Part 3: Handling collisions with other objects

## Part 4: Handling self-collisions
<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
    <tr>
      <td align="center">
        <video controls="controls" width="100%" name="Video Name">
            <source src="../assets/hw4/part4/pre_self_collision.mov">
        </video>
        <figcaption>send help</figcaption>
      </td>
      <td align="center">
        <video controls="controls" width="100%" name="Video Name">
            <source src="../assets/hw4/part4/vid.mov">
        </video>
        <figcaption>send help</figcaption>
      </td>
    </tr>
  </table>
</div>

## Part 5: Shaders

## Part 6: Extra credit

## Contributors
Edward Park, Ashley Chiu
TODO