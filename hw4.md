---
layout: page
title: 'Homework 4: Clothsim'
has_right_toc: true
usemathjax: true
---
<p class="warning-message">
This assignment has not been completed yet.
</p>

site: [https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw4/](https://cal-cs184-student.github.io/hw-webpages-sp24-ashmchiu/hw4/)

## Overview
In this assignment, we create a physical simulation of a cloth. The cloth is able to maintain its shape and behavior through internal spring forces. It is also able to collide with external objects and itself. We also explore different shading options for the cloth, which includes diffuse and Phong shading, texture mapping, bump and displacement mapping, and ideal specular (mirror-like) environment mapping using cubemaps. Finally, we implement a few extra features, such as time-varying wind forces, a transparent blue-tinted shader and oscillating vertex shifter (to mimic a bouncing ball), and collisions with a new 3D primitive: cubes.

What we found interesting was the differing techniques for intersecting cloths with different primitives. In hindsight, it makes sense why we needed to perform collisions with them in different ways. It was fun to see all the different shaders in [Part 5](/hw4.md#part-5-shaders) come together--specifically, the texture map was really fun to add new textures into! It was also fun to see how this homework related back to [Homework 2](/hw2.md) with Phong shading.

We actually didn't have difficult debugging journeys in this homework. Namely, something we were stuck on for a long time was [bump and displacement mapping](/hw4.md#task-4-displacement-and-bump-mapping) because our renders had much softer bumps and displacements than the references. However, we realized that this is because the default `normal` was `2` and `height` was `0.1` and when we updated this to be a `normal` of `100` and a `height` of `0.061`, we could more clearly see the bumps and displacements.

## Part 1: Masses and springs
In this part, our main goal was creating a grid of point masses and springs. To do so, we iterated through <code class="language-plaintext highlighter-rouge">num_height_points</code> and an inner loop of <code class="language-plaintext highlighter-rouge">num_width_points</code> to generate our point masses in row-major order. Depending on whether the orientation was horizontal or vertical, we either varied across the <code class="language-plaintext highlighter-rouge">xz</code> plane or the <code class="language-plaintext highlighter-rouge">xy</code> plane. Furthermore, if the point mass's <code class="language-plaintext highlighter-rouge">(x, y)</code> index was within the cloth's <code class="language-plaintext highlighter-rouge">pinned</code> vector, then we set their <code class="language-plaintext highlighter-rouge">pinned</code> boolean to <code class="language-plaintext highlighter-rouge">true</code> (which we'll see at the corners of ../scene/pinned4.json).

Now that we've created our grid of point masses, we then created our springs with structural, shearing, and bending constraints. We note that
- structural constraints were applied between a point mass and the point masses to its left and directly above it (if they existed)
- shearing constraints were applied between a point mass and the point masses to its diagonal upper left and right (if they existed)
- bending constraints were applied between a point mass and the point masses two to the left and two above it (if they existed).

Below, we've included several screenshots of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/pinned2.json</code> from various viewing angles to show the wireframe and structure of point masses and springs.
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
        <img src="../assets/hw4/part1/all.png" width="50%"/>
        <figcaption>../scene/pinned2.json, with all constraints</figcaption>
    </td>
    </tr>
  </table>
</div>

## Part 2: Simulation via numerical integration
Our main goal in this part is completing the <code class="language-plaintext highlighter-rouge">Cloth::simulate</code> method that runs one time step of time length <code class="language-plaintext highlighter-rouge">delta_t</code> and applies all <code class="language-plaintext highlighter-rouge">accelerations</code> uniformly to all point masses in the cloth.

### Task 1: Compute total force acting on each point mass
First, we calculated all the external forces based on the <code class="language-plaintext highlighter-rouge">external_accelerations</code>, which just contains <code class="language-plaintext highlighter-rouge">gravity</code> as of now (see our [wind](/hw4.md#whoosh-wind) simulations for an updated <code class="language-plaintext highlighter-rouge">external_accelerations</code>). Since we're applying all accelerations uniformly, we can just sum them once, using Newton's 2nd Law, $$F = ma$$ (noting that the $$m$$, mass, is constant). Then, we set each point mass's <code class="language-plaintext highlighter-rouge">forces</code> to a <code class="language-plaintext highlighter-rouge">Vector3D</code> of our summed external forces.

Then, we needed to apply spring correction forces (based on our work in [Part 1](/hw4.md#part-1-masses-and-springs) in assigning spring constraints). We used Hooke's law, $$F_s = k_s * (\vert\vert p_a - p_b\vert\vert - l)$$, such that for each spring, we would apply $$F_s$$ force to the point mass on one end, and then an equal, but opposite (negated) force to the other. For bending constraints, we set $$F_s$$ to $$0.2$$x compared to shearing and structural constraints since they're normally weaker.

### Task 2: Use Verlet integration to compute new point mass positions
Next, we then computed a point mass's new position since we now know the force acting on each point mass at a specific time step. For this, we use Verlet integration on all un-pinned vertices, calculating updated positions via

$$
x_{t + dt} = x_t + (1 - d) * (x_t - x_{t - dt}) + a_t * dt^2
$$

noting that $$d$$ serves as a <code class="language-plaintext highlighter-rouge">damping</code> term given by the <code class="language-plaintext highlighter-rouge">ClothParameters</code>. We made sure in this step to also update the point mass's <code class="language-plaintext highlighter-rouge">last_position</code> to track positions through time.

### Task 3: Constrain position updates
Finally, to keep springs from being unreasonably deformed, we used the [SIGGRAPH 1995 Provot paper](https://www.cs.rpi.edu/~cutler/classes/advancedgraphics/S14/papers/provot_cloth_simulation_96.pdf) to prevent springs from extending past than 10% of their <code class="language-plaintext highlighter-rouge">rest_length</code> at the end of any time step. If they did, we would correct the positions of the spring's point masses:
- If neither of the point masses were pinned, then we would performed half of the correct to each point mass.
- If one of the point masses was pinned, we corrected fully by the other point mass.
- If both of the point masses were pinned, we did nothing (because they couldn't be moved :')).

Below, we've included screenshots of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/pinned4.json</code> with default parameters, with both the wireframe normal appearance.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/wireframe.png" width="100%"/>
      <figcaption>../scene/pinned4.json, final resting state, wireframe</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/normal.png" width="100%"/>
      <figcaption>../scene/pinned4.json, final resting state, normal</figcaption>
    </td>
  </tr>
  </table>
</div>

### Experimenting with parameters
In our cloth simulator, we have the ability to change the spring constant <code class="language-plaintext highlighter-rouge">ks</code>, the <code class="language-plaintext highlighter-rouge">density</code>, and <code class="language-plaintext highlighter-rouge">damping</code> constants. We'll describe how the cloth differs when changing these to the default parameters. Below is the wireframe and normal appearance of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/pinned2.json</code> to show default parameters (<code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code>, <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code>, <code class="language-plaintext highlighter-rouge">damping = 0.200000%</code>).

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/pinned2_wireframe.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> default parameters</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/pinned2_normal.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br> default parameters</figcaption>
    </td>
  </tr>
  </table>
</div>

#### Changing <code class="language-plaintext highlighter-rouge">ks</code>
While maintaining the default <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code> and <code class="language-plaintext highlighter-rouge">damping = 0.200000%</code>, let's show ../scene/pinned2.json with <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, <code class="language-plaintext highlighter-rouge">ks = 500 N/m</code>, and <code class="language-plaintext highlighter-rouge">ks = 50000 N/m</code>.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/ks/pinned2_wireframe_ks50.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, default <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/ks/pinned2_normal_ks50.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br><code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, default <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/ks/pinned2_wireframe_ks500.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">ks = 500 N/m</code>, default <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/ks/pinned2_normal_ks500.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br> <code class="language-plaintext highlighter-rouge">ks = 500 N/m</code>, default <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/ks/pinned2_wireframe_ks50000.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, default <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/ks/pinned2_normal_ks50000.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br> <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, default <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  </table>
</div>

As <code class="language-plaintext highlighter-rouge">ks</code> increases, we see that the strength of the springs increase, which means that they bend much less and stay much flatter than when <code class="language-plaintext highlighter-rouge">ks</code> is smaller. Namely, this means that at our highest example of <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, when the cloth is resting, there is little creasing (beyond to account for the pinning). Yet, in contrast, we see that when <code class="language-plaintext highlighter-rouge">ks</code> is smaller, the cloth is much more free and has many creases and folds at rest position. Since it's much more free to move, even at rest position, with <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, it is loose and wiggles: it doesn't stay completely still.

When comparing how the cloth behaves as it moves from start to rest, we note that at lower <code class="language-plaintext highlighter-rouge">ks</code> values, because the spring constant is much less, the springs are less tight, so the cloth is a lot more flexible and waves as it swings down. In comparison, with higher <code class="language-plaintext highlighter-rouge">ks</code> values, the cloth is much more rigid and stays relatively flat as it swings down to rest position.

#### Changing <code class="language-plaintext highlighter-rouge">density</code>
While maintaining the default <code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code> and <code class="language-plaintext highlighter-rouge">damping = 0.200000%</code>, let's show ../scene/pinned2.json with <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, <code class="language-plaintext highlighter-rouge">density = 500 g/cm^2</code>, and <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_wireframe_density1.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_normal_density1.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br><code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_wireframe_density50.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_normal_density50.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br><code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_wireframe_density500.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">density = 500 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_normal_density500.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br> <code class="language-plaintext highlighter-rouge">density = 500 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_wireframe_density5000.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/density/pinned2_normal_density5000.png" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br> <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">damping</code></figcaption>
    </td>
  </tr>
  </table>
</div>

We see that <code class="language-plaintext highlighter-rouge">density</code> operates almost inversely to <code class="language-plaintext highlighter-rouge">ks</code>. Namely, at the lowest <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, we see the most rigid cloth, where there are less deformations in the cloth. Because at lower densities, the cloths will have lower mass, this means that the forces (that we accumulate) at each point mass will be less, and thus, means that there's less forces pulling the cloth down, causing less wrinkles. In contrast, at our highest <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>, there are larger external forces working on the point masses (due to Newton's 2nd Law). This means that the cloth weighs more, and thus, has many more wrinkles, which we can visibly see in the normal appearance with <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>.

When discussing how the cloth behaves as it moves from start to rest, we note that lower <code class="language-plaintext highlighter-rouge">density</code> values, the cloth weighs less, and as such, stays relatively flat as the forces acting against it aren't as strong. This differs from using higher <code class="language-plaintext highlighter-rouge">density</code> values as the forces acting on the cloth are now stronger, and thus, the cloth deforms more, which causes more waves and wrinkles as it swings down.

#### Changing <code class="language-plaintext highlighter-rouge">damping</code>

While maintaining the default <code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code> and <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code>, let's show ../scene/pinned2.json with <code class="language-plaintext highlighter-rouge">damping = 0%</code> and <code class="language-plaintext highlighter-rouge">damping = 1%</code>.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/damping/pinned2_wireframe_damping0.png" width="100%"/>
      <figcaption>../scene/pinned2.json, moving, wireframe <br> <code class="language-plaintext highlighter-rouge">damping = 0%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/damping/pinned2_normal_damping0.png" width="100%"/>
      <figcaption>../scene/pinned2.json, moving, normal <br><code class="language-plaintext highlighter-rouge">damping = 0%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/damping/pinned2_wireframe_damping1.png" width="100%"/>
      <figcaption>../scene/pinned2.json, moving, wireframe <br> <code class="language-plaintext highlighter-rouge">damping - 1%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/damping/pinned2_normal_damping1.png" width="100%"/>
      <figcaption>../scene/pinned2.json, moving, normal <br><code class="language-plaintext highlighter-rouge">damping = 1%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
  </tr>
  </table>
</div>

Here, we've opted to also include .gif depictions because what we believe to be more important is demonstrating the speed and flexibility at which the cloth moves. 

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/damping/wireframe_low_damping.gif" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">damping = 0%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/damping/normal_low_damping.gif" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br><code class="language-plaintext highlighter-rouge">damping = 0%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part2/damping/wireframe_high_damping.gif" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, wireframe <br> <code class="language-plaintext highlighter-rouge">damping = 1%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part2/damping/normal_high_damping.gif" width="100%"/>
      <figcaption>../scene/pinned2.json, at rest, normal <br><code class="language-plaintext highlighter-rouge">damping = 1%</code>, default <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code></figcaption>
    </td>
  </tr>
  </table>
</div>

Finally, we compare what happens when we mess with  <code class="language-plaintext highlighter-rouge">damping</code>. At the lowest possible <code class="language-plaintext highlighter-rouge">damping = 0%</code> provided, we see that the cloth swings quite quickly back and forth. In contrast, at the highest possible  <code class="language-plaintext highlighter-rouge">damping = 1%</code>, the cloth moves really quickly and seemingly stops at the rest position (that matches the rest position with default parameters). Since we know that damping messes with the velocity term in Verlet integration, with no damping, this means there is no loss of energy due to friction, heat loss, or any other force. Therefore, the cloth just swings back and forth, with wrinkling and no rigid structure because there is no loss of energy. However, with the highest damping, the cloth falls much slower and holds its structure a bit better. The expalanation for this is that the forces acting on it (namely, gravity) are now dampened, which means that the positions of the point masses don't move as quickly. This reflects a state in which there is a loss of energy, so not all the forces that are thrust upon the cloth actually convert into energy that moves the cloth--some is lost or dissipated.

## Part 3: Handling collisions with other objects
Throughout this part, we are colliding a cloth with [spheres](/hw4.md#task-1-handling-collisions-with-spheres) and [planes](/hw4.md#task-2-handling-collisions-with-planes). We want to make the cloth collide in a realstic manner--which we will later elevate in [Part 4](/hw4.md#part-4-handling-self-collisions) with self collisions!

### Task 1: Handling collisions with spheres
In handling sphere collisions, we implemented <code class="language-plaintext highlighter-rouge">Sphere::collide</code>, noting that if the position of the point mass would be within the sphere, we would "bump" it to the surface of the sphere. To do so, we first used Euclidean distances to check whether the point mass was inside the sphere, returning if it wasn't. Then, we computed the tangent point along the sphere where the collision would have occurred, using this to compute the correction vector and applying that correction vector to the point mass's <code class="language-plaintext highlighter-rouge">last_position</code>, scaled by <code class="language-plaintext highlighter-rouge">1 - f</code> where <code class="language-plaintext highlighter-rouge">f</code> is friction.

We also updated <code class="language-plaintext highlighter-rouge">Cloth::simulate</code>, checking for collisions between every <code class="language-plaintext highlighter-rouge">PointMass</code> and every possible <code class="language-plaintext highlighter-rouge">CollisionObject</code>, of which <code class="language-plaintext highlighter-rouge">Sphere</code> was a type.

Below, we've included screenshots of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> in its final resting state, using the default <code class="language-plaintext highlighter-rouge">ks = 5000</code>, as well as <code class="language-plaintext highlighter-rouge">ks = 500</code> and <code class="language-plaintext highlighter-rouge">ks = 50000</code>.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part3/sphere_wireframe.png" width="100%"/>
      <figcaption>../scene/sphere.json, final resting state, wireframe, <br>default <code class="language-plaintext highlighter-rouge">ks = 5000</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part3/sphere_normal.png" width="100%"/>
      <figcaption>../scene/sphere.json, final resting state, normal, <br>default <code class="language-plaintext highlighter-rouge">ks = 5000</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part3/sphere_wireframe_500.png" width="100%"/>
      <figcaption>../scene/sphere.json, final resting state, wireframe, <br><code class="language-plaintext highlighter-rouge">ks = 500</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part3/sphere_normal_500.png" width="100%"/>
      <figcaption>../scene/sphere.json, final resting state, normal, <br><code class="language-plaintext highlighter-rouge">ks = 500</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part3/sphere_wireframe_50000.png" width="100%"/>
      <figcaption>../scene/sphere.json, final resting state, wireframe, <br><code class="language-plaintext highlighter-rouge">ks = 50000</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part3/sphere_normal_50000.png" width="100%"/>
      <figcaption>../scene/sphere.json, final resting state, normal, <br><code class="language-plaintext highlighter-rouge">ks = 5000</code></figcaption>
    </td>
  </tr>
  </table>
</div>

We see that when <code class="language-plaintext highlighter-rouge">ks</code> increases, the cloth becomes more rigid and the folds structured and hence, there are also less folds. In contrast, when <code class="language-plaintext highlighter-rouge">ks</code> decreases, the cloth becomes much more flexible, and we can see that the folds around the sphere increase as the cloth as able to mold to the sphere more. We can see that at lower <code class="language-plaintext highlighter-rouge">ks</code> values, the cloth drapes more willingly, so we can see the spherical structure more whereas at the highest <code class="language-plaintext highlighter-rouge">ks</code> value, the creasing at the top obscures the sphere's smoothness around the sides.

In discussion regarding the purpose of <code class="language-plaintext highlighter-rouge">ks</code>, this does make sense as a stronger/larger spring constant means that the spring is stronger, so the cloth holds shape stronger (and vice versa for the smaller <code class="language-plaintext highlighter-rouge">ks</code>).

### Task 2: Handling collisions with planes
To handle collisions with planes, we implemented the <code class="language-plaintext highlighter-rouge">Plane::collide</code> method. Similarly, we first checked whether a point had crossed the plane by checking whether the dot product between the normal vector of the plane and the point mass's <code class="language-plaintext highlighter-rouge">last_position</code> and <code class="language-plaintext highlighter-rouge">position</code> had different signs. If the signs were the same, this meant that the point mass had not passed the plane, so we directly returned.

Otherwise, we again calcualted the tangent point at which the collision between the point mass and the plane would have occurred, using that to compute the correction vector (in which we factored in the <code class="language-plaintext highlighter-rouge">SURFACE_OFFSET</code> by the <code class="language-plaintext highlighter-rouge">normal</code> vector of the <code class="language-plaintext highlighter-rouge">Plane</code>). Then, similarly to <code class="language-plaintext highlighter-rouge">Sphere::collide</code>, we applied the correction vector to the point mass's <code class="language-plaintext highlighter-rouge">last_position</code>, scaled by <code class="language-plaintext highlighter-rouge">1 - f</code> where <code class="language-plaintext highlighter-rouge">f</code> is friction.

We also updated <code class="language-plaintext highlighter-rouge">Cloth::simulate</code>, checking for collisions between every <code class="language-plaintext highlighter-rouge">PointMass</code> and every possible <code class="language-plaintext highlighter-rouge">CollisionObject</code>, of which <code class="language-plaintext highlighter-rouge">Plane</code> was a type.

Here's a few screenshots of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/plane.json</code>, lying peacefully at rest on the plane. (We are kalm -- no panic!)

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part3/plane_wireframe.png" width="100%"/>
      <figcaption>../scene/plane.json, final resting state, wireframe</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part3/plane_normal.png" width="100%"/>
      <figcaption>../scene/plane.json, final resting state, normal</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part3/plane_mirror.png" width="100%"/>
      <figcaption>../scene/plane.json, final resting state, mirror</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part3/plane_eddie_proctor_vidoe.png" width="100%"/>
      <figcaption>../scene/plane.json, final resting state, custom texture (eddie)</figcaption>
    </td>
  </tr>
  </table>
</div>

## Part 4: Handling self-collisions
In [Part 3](/hw4.md#part-3-handling-collisions-with-other-objects), we handled collisions between a cloth and other objects, however, if a cloth were to collide with itself right now, it would have no comprehension of that collision (and just clip through itself). What we aim to implement (and what we did!) was prevent this clipping, ensuring that if the cloth fell on itself, it would fold. Below, we run <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/selfCollision.json</code> prior to implementing Task 4 and after. To the left, you can see the clipping, particularly the glitching after the cloth is fully on the plane (no longer resting--this is a panic!). To the right, you can see the cloth folding over itself and resting in a much more peaceful way.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
    <tr>
      <td align="center">
        <img src="../assets/hw4/part4/pre_self_collision.gif" width="100%"/>
        <figcaption>A very send help moment.</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw4/part4/self_collision.gif">
        <figcaption>Help was sent.</figcaption>
      </td>
    </tr>
  </table>
</div>

To perform this task, we implemented spatial hashing, mapping floats to a <code class="language-plaintext highlighter-rouge">vector<PointMass *></code>, where each float uniquely represented a 3D box volume in the scene and the <code class="language-plaintext highlighter-rouge">vector<PointMass *></code> was a vector containing all the point masses in that 3D box volume. Using this hash table, we would apply a repulsive collision force if any pair of point masses in the same 3D box volume got too close to one another.

### Task 1: <code class="language-plaintext highlighter-rouge">Cloth::hash_position</code>
First, we implemented <code class="language-plaintext highlighter-rouge">Cloth::hash_position</code> to take in a point mass's position and to effectively calculate its 3D box volume's index in our spatial hash table. To do so, we calculated a <code class="language-plaintext highlighter-rouge">(w, h, t)</code> such that <code class="language-plaintext highlighter-rouge">w = 3 * width / num_width_points</code>, <code class="language-plaintext highlighter-rouge">h = 3 * height / num_height_points</code>, and <code class="language-plaintext highlighter-rouge">t = max(w, h)</code>. From here, we determined what 3D box the point was in by calculating <code class="language-plaintext highlighter-rouge">x = floor(pos.x / w) * w</code>, and similarly for the <code class="language-plaintext highlighter-rouge">y</code> and <code class="language-plaintext highlighter-rouge">z</code> axes. Knowing this, we transformed this 3D position into a 1D position by using a formmulation similar to how we determined indices for supersampled points in [Homework 1](/hw1.md#resizing-the-sample-buffer), returning this value.

### Task 2: <code class="language-plaintext highlighter-rouge">Cloth::build_spatial_map</code>
Now, since we had a way of calculating each point mass's hash position, we can now construct our spatial map. To do so, we iterated over all <code class="language-plaintext highlighter-rouge">point_masses</code> in the cloth and found its [<code class="language-plaintext highlighter-rouge">hash_position</code>](/hw4.md#task-1-clothhash_position). Then, since at each key-value pair in the map, our value is a <code class="language-plaintext highlighter-rouge">vector<PointMass *></code>, we initialize a new <code class="language-plaintext highlighter-rouge">vector<PointMass *></code> if there isn't already one at the hash position, and then we <code class="language-plaintext highlighter-rouge">push_back</code> the current point mass.

### Task 3: <code class="language-plaintext highlighter-rouge">Cloth::self_collide</code>
Finally, we implemented our full self-collision method. For the passed in point mass, we check whether the normalized distance between it and all candidate point masses is less than $$2 * thickness$$, and also whether it's normalized distance is greater than 0 (to prevent a point mass from colliding with itself). Then, we would calculate the final correction to be unit distance between the two point masses multipled by the correction to ensure that the pair would be $$2 * thickness$$ distance apart.

Then, we updated <code class="language-plaintext highlighter-rouge">Cloth::simulate</code>, first to call our <code class="language-plaintext highlighter-rouge">build_spatial_map</code> function. Then, we iterated through all <code class="language-plaintext highlighter-rouge">point_masses</code>, calling <code class="language-plaintext highlighter-rouge">self_collide</code> on it.

Below are 6 screenshots of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/selfCollision.json
</code> to show how the cloth falls and folds on itself.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/1early.png" width="100%"/>
      <figcaption>../scene/plane.json, early, initial self-collision</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/2middle.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, continuing folding over itself</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/3squid.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, deep into self-collisions</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/4flattening.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, beginning to flatten out</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/5unfurl.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, smoothing out</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/6flatish.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, final resting state</figcaption>
    </td>
  </tr>
  </table>
</div>

Now, we're going to be experimenting with <code class="language-plaintext highlighter-rouge">ks</code> and <code class="language-plaintext highlighter-rouge">density</code> values. As a reference, here is a .gif of the self-collision in <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/selfCollision.json</code> with default <code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code> and <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code>.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part4/self_collision_default.gif" width="50%"/>
        <figcaption>../scene/selfCollision.json, <br>default <code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code>, <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code></figcaption>
      </td>
    </tr>
  </table>
</div>

### Experimenting with <code class="language-plaintext highlighter-rouge">ks</code>

Here are .gif files of the self-collision in <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/selfCollision.json</code> with default <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code>, but varying <code class="language-plaintext highlighter-rouge">ks</code> with a low <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code> and a high <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>.
<div>
<table>
  <tr>
      <td align="center">
        <img src="../assets/hw4/part4/self_collision_low_ks.gif" width="100%"/>
        <figcaption>../scene/selfCollision.json, low <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw4/part4/self_collision_high_ks.gif" width="100%"/>
        <figcaption>../scene/selfCollision.json, high <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code></figcaption>
      </td>
    </tr>
  </table>
</div>

We can see through these .gif files and the below screenshots that at smaller <code class="language-plaintext highlighter-rouge">ks</code> values, such as the <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code> provided, the cloth folds in a much more rippling fashion, with each fold being smaller. This allows overall for more self-collisions, because at a lower spring constant, the cloth doesn't hold as much rigid structure, so it is flexible to fold a lot. In contrast, with a higher <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, we see that since the spring constant is higher, the cloth is tighter and more rigid, holding its structure, which means that it folds a lot less against itself, so there are less self-collisions and overall, less wrinkles. There are gaps between the layers of the cloth as it lays on top of other parts of itself, which we can see more in the second and third screenshots for <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>.

To the left, we've included screenshots of how the cloth behaves with <code class="language-plaintext highlighter-rouge">ks = 50 N/m</code> as it falls on itself while on the right are screenshots of how the cloth behaves with <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>.

<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50_1.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, beginning to ripple</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50000_1.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, small waves</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50_2.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, rippling more</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50000_2.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, more small waves</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50_3.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, beginning to settle</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50000_3.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, beginning to settle</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50_4.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">ks = 50 N/m</code>, resting state</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/ks_50000_4.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <code class="language-plaintext highlighter-rouge">ks = 50,000 N/m</code>, resting state</figcaption>
    </td>
  </tr>
  </table>
</div>

### Experimenting with <code class="language-plaintext highlighter-rouge">density</code>

Here are .gif files of the self-collision in <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/selfCollision.json</code> with default <code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code>, but varying <code class="language-plaintext highlighter-rouge">density</code> with a low <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code> and a high <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>.
<div>
<table>
  <tr>
      <td align="center">
        <img src="../assets/hw4/part4/self_collision_low_density.gif" width="100%"/>
        <figcaption>../scene/selfCollision.json, low <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw4/part4/self_collision_high_density.gif" width="100%"/>
        <figcaption>../scene/selfCollision.json, high <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code></figcaption>
      </td>
    </tr>
  </table>
</div>

We can see through these .gif files and the below screenshots that at smaller <code class="language-plaintext highlighter-rouge">density</code> values, such as the <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code> provided, the cloth almost seems bouncier, holding its structure and making larger waves. This means there are less self-collisions: this makes sense because at a lower density, this means that the mass is less, so the overall force applied to the cloth is less, making it collide less. In contrast, with a higher <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, we see that the cloth folds a lot more, rippling as it falls on itself. Compared to smaller densities, each fold in the cloth is smaller. At a higher density, this makes sense because larger density means larger mass, and as such, there is a larger force applied to it, allowing for more self-collisions.

To the left, we've included screenshots of how the cloth behaves with <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code> as it falls on itself while on the right are screenshots of how the cloth behaves with <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>.

<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/density_1_1.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, beginning to ripple</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/density_50_1.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, small waves</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/density_1_2.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, rippling more</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/density_50_2.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, more small waves</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/density_1_3.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, beginning to settle</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/density_50_3.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, beginning to settle</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part4/density_1_4.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <br><code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, resting state</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part4/density_50_4.png" width="100%"/>
      <figcaption>../scene/selfCollision.json, <code class="language-plaintext highlighter-rouge">density = 50 g/cm^2</code>, resting state</figcaption>
    </td>
  </tr>
  </table>
</div>

## Part 5: Shaders
TODO: Explain in your own words what is a shader program and how vertex and fragment shaders work together to create lighting and material effects.

### Task 1: Diffuse Shading
Implementing the <code class="language-plaintext highlighter-rouge">main</code> method of <code class="language-plaintext highlighter-rouge">Diffuse.frag</code>, our main goal was to recreate diffuse lighting using the formula

$$
L_d = k_d (I/r^2)\max(0, n \cdot l)
$$

We did this by calculating the radius from the light position to the vertex, calculating the normalized normal vector, and normalizing the light vector. From there, we set the <code class="language-plaintext highlighter-rouge">out_color</code>, noting that we are given $$k_d = 1$$ to $$L_d$$ as given in the diffuse lighting formula.

Below is a screenshot of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> with diffuse shading.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part5/diffuse.png" width="50%"/>
        <figcaption>../scene/sphere.json, diffuse shading</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 2: Blinn-Phong Shading
From lecture, we know that Blinn-Phong shading outputs a light with the equation

$$
L = k_a I_a + k_d (I/r^2) \max(0, n \cdot l) + k_s (I/r^2) \max(0, n \cdot h)^p
$$

effectively, the addition of ambient, diffuse, and specular lighting (which reflect each of the three terms in the equation above). We note that ambient lighting is shading that doesn't depend on anything (it is constant). As a whole, Blinn-Phong shading allows us to account for not only the angle between each vertex and a light source (diffuse), but also from what perspective and viewing angle the camera is from (specular). Namely, the coolest part of Blinn-Phong shading (in our opinion), is that in specular shading, it uses the half-vector between the viewing direction and the light source, which is used to approximate where the maximum specular reflection (the greatest intensity reflection) is. Blinn-Phong is faster than the methods we took in [Part 3](/hw3.md) to light up scenes.

Building off our work in  <code class="language-plaintext highlighter-rouge">Diffuse.frag</code>, we also now need to compute ambient and specular shading.

For ambient shading, we choose <code class="language-plaintext highlighter-rouge">k_a = 1</code> and  <code class="language-plaintext highlighter-rouge">I_a = vec(1/61, 1/161, 1/61)</code>. Since the formula is $$L_a = k_a I_a$$, this is all we need! With only purely ambient shading, running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> gives

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part5/phong_ambient.png" width="50%"/>
        <figcaption>../scene/sphere.json, ambient shading</figcaption>
      </td>
    </tr>
  </table>
</div>

Then, we reused our code from <code class="language-plaintext highlighter-rouge">Diffuse.frag</code> for diffuse shading, $$L_d$$. With only purely diffuse shading, running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> gives

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part5/phong_diffuse.png" width="50%"/>
        <figcaption>../scene/sphere.json, diffuse shading</figcaption>
      </td>
    </tr>
  </table>
</div>

Finally, we add our specular highlights! To do so, we calculate the viewing vector <code class="language-plaintext highlighter-rouge">v</code> as the difference between the camera position and the vertex position. We then calculate the half-vector between <code class="language-plaintext highlighter-rouge">v</code> and the normal vector, normalizing the half-vector. Finally, we calculate our specular shading using the formula $$L_s = k_s (I / r^2) * \max(0, n \cdot h)^p$$. We use $$k_s = 1$$ and $$p = 161$$. With only purely specular shading, running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> gives

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part5/phong_specular.png" width="50%"/>
        <figcaption>../scene/sphere.json, specular shading</figcaption>
      </td>
    </tr>
  </table>
</div>

We see the main specular point near the center and front of the sphere.

Now, let's put it all together. We know that Blinn-Phong reflections are just a summation of ambient, diffuse, and specular shading. Therefore, we get our resulting $$L = L_a + L_d + L_s$$, which when running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code>, gives

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part5/phong.png" width="50%"/>
        <figcaption>../scene/sphere.json, entire Blinn-Phong model</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 3: Texture Mapping
Modifying <code class="language-plaintext highlighter-rouge">Texture.frag</code>, we set the output single-4 dimensional vector to be a call to the built-in function <code class="language-plaintext highlighter-rouge">texture(sampler2D tex, vec2 uv)</code>, passing in the texture map provided and the texture space coordinate <code class="language-plaintext highlighter-rouge">v_uv</code>.

Below, we include screenshots of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> with the default Campanile texture map and :eddie-proctor-vidoe:, one of our favorite Slackmojis.

<div align="center">
  <table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/texture.png" width="100%"/>
      <figcaption>../scene/sphere.json, default Campanile texture</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/texture_eddie_proctor_vidoe.png" width="100%"/>
      <figcaption>../scene/sphere.json, custom texture</figcaption>
    </td>
  </tr>
  </table>
</div>

For reference, here is :eddie_proctor_vidoe: (yes, the spelling is correct) in all its glory:
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/part5/eddie_proctor_vidoe.png" width="350px"/>
        <figcaption>:eddie_proctor_vidoe:</figcaption>
      </td>
    </tr>
  </table>
</div>

### Task 4: Displacement and Bump Mapping
In this task, we implement both displacement and bump mapping, two different ways of representing surface roughness. The first part of this task is bump mapping, where the texture map stores height information. The height information gleaned from the texture map is used to calculate the displaced model space normal $n_d$ from the original normal vector inside <code class="language-plaintext highlighter-rouge">Bump.frag</code>.

For bump mapping, in <code class="language-plaintext highlighter-rouge">Bump.frag</code>, we implemented our <code class="language-plaintext highlighter-rouge">h(vec2 uv)</code> function to return the r-value of the return of calling <code class="language-plaintext highlighter-rouge">texture</code>. We also used the provided <code class="language-plaintext highlighter-rouge">normal</code> and <code class="language-plaintext highlighter-rouge">tangent</code> to calculate the <code class="language-plaintext highlighter-rouge">bitangent = cross(normal, tangent)</code> to create the <code class="language-plaintext highlighter-rouge">TBN</code> matrix as

$$TBN = \begin{bmatrix}t & b & n\end{bmatrix}$$ 

The displaced model space normal is calculated by first calculating the differential height via

$$dU = (h(u + \frac{1}{w}, v) - h(u, v)) \times k_h \times k_n$$

$$dV = (h(u, v + \frac{1}{h}) - h(u, v) \times k_h \times k_n)$$

where $k_h$ and $k_n$ are height and normal scaling factors, respectively, and $h$ is a function that returns the value of the texture map at the passed in coordinates.

Finally, we can apply the TBN matrix to the local space normal $n_o = (-dU, -dV, 1)$ to get $n_d$. This result is like calculating the normal vector to the object had its surface been displaced by the texture map height location. Importantly, though, the surface is not displaced in bump mapping, and the object maintains its original geometry. We reuse our code from <code class="language-plaintext highlighter-rouge">Phong.frag</code> as our shading calculations, replacing our <code class="language-plaintext highlighter-rouge">normal</code> vector with our calculated $$n_d$$.

Below are screenshots of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> with bump mapping, setting the normal to 100 and height to 0.061. We use <code class="language-plaintext highlighter-rouge">texture3.png</code>.

<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/bump_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, bump mapping on sphere</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/bump.png" width="100%"/>
      <figcaption>../scene/sphere.json, bump mapping on sphere and cloth</figcaption>
    </td>
  </tr>
  <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw4/part5/bump_cloth_sphere.png" width="50%"/>
        <figcaption>../scene/sphere.json, bump mapping with cloth over sphere</figcaption>
      </td>
    </tr>
  </table>
</div>

For displacement mapping, we copied over the work we did in <code class="language-plaintext highlighter-rouge">Bump.frag</code> into <code class="language-plaintext highlighter-rouge">Displacement.frag</code>. Then, we updated <code class="language-plaintext highlighter-rouge">Displacement.vert</code> to calculate <code class="language-plaintext highlighter-rouge">v_position</code> as the old position <code class="language-plaintext highlighter-rouge">u_model * in_position</code> summed with the original model space vertex normal <code class="language-plaintext highlighter-rouge">normalize(u_model * in_normal)</code> scaled by <code class="language-plaintext highlighter-rouge">h(in_uv) * u_height_scaling</code>. Because <code class="language-plaintext highlighter-rouge">gl_position</code> represents the screen space position, we also modified this to reflect the new position.

Below are screenshots of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> with displacement mapping, setting the normal to 100 and height to 0.061. We use <code class="language-plaintext highlighter-rouge">texture3.png</code>.

<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/displacement_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, displacement mapping on sphere</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/displacement.png" width="100%"/>
      <figcaption>../scene/sphere.json, displacement mapping on sphere and cloth</figcaption>
    </td>
  </tr>
  <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw4/part5/displacement_cloth_sphere.png" width="50%"/>
        <figcaption>../scene/sphere.json, displacement mapping with cloth over sphere</figcaption>
      </td>
    </tr>
  </table>
</div>

#### Comparing Bump and Displacement Mapping
<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/bump_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, bump mapping on sphere</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/displacement_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, displacement mapping on sphere</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/bump_cloth_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, bump mapping with cloth over sphere</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/displacement_cloth_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, displacement mapping with cloth over sphere</figcaption>
    </td>
  </tr>
  </table>
</div>

Looking at these images where the only difference is using bump mapping versus displacement mapping, we see that bump mapping maintains the smoothness of the sphere while displacement mapping, while actually changing the location of each vertex, no longer is smooth. Namely, we see that bump mapping mainly modifies the fragments, not the locations of the vertices themselves across screen space, so it gives an illusion of bumps and highlights and indentations, by creating highlights where the brick highlights would be through the reuse of [Phong shading](/hw4.md#task-2-blinn-phong-shading). On the other hand, we see that displacement mapping actually moves around the vertices of the sphere based on the texture map , matching the roughness of the bricks, deforming from the perfect smooth texture the sphere had.

Playing around with coarseness, here are screenshots of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> with bump displacement mapping, setting the normal to 100 and height to 0.061. We use <code class="language-plaintext highlighter-rouge">texture3.png</code>. On the top row, we use <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json -o 16 -a 16</code> for lower resolution and on the bottom row, we use <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json -o 128 -a 128</code> for higher resolution.

<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/bump_o16_a16.png" width="100%"/>
      <figcaption>../scene/sphere.json, bump mapping <br><code class="language-plaintext highlighter-rouge">-o 16 -a 16</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/displacement_o16_a16.png" width="100%"/>
      <figcaption>../scene/sphere.json, displacement mapping <br><code class="language-plaintext highlighter-rouge">-o 16 -a 16</code></figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/bump_o128_a128.png" width="100%"/>
      <figcaption>../scene/sphere.json, bump mapping <br><code class="language-plaintext highlighter-rouge">-o 128 -a 128</code></figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/displacement_o128_a128.png" width="100%"/>
      <figcaption>../scene/sphere.json, displacement mapping <br><code class="language-plaintext highlighter-rouge">-o 128 -a 128</code></figcaption>
    </td>
  </tr>
  </table>
</div>

Now, when we compare the coarseness of the sphere while changing the resolution of the image, the displacement mapping affects the coarseness of the sphere more than bump mapping does. Namely, we see that for the smaller resolution image (lower coarseness), there's less sampling points across the sphere for the displacement map to change the vertices of, so we can clearly see the jumps across different vertices (there are extreme jagged edges across the sphere and it is more spikey than it is smooth). In contrast, at a higher resolution, the coarseness is higher, and the surface texture of the sphere is much sharper with displacement mapping. More vertices have modified positions due to the displacement mapping so the texture seems more realistic and reflects the texture of the brick better. Because there are more vertices whose positions are displaced, although the displacement mapping sphere is still spikey, the overall roundness of the sphere is maintained.

While displacement mapping has large differences across different resolutions, bump mapping does not. Because bump mapping doesn't actually change the position of vertices (thus, changing the shape of the sphere), and instead just maps the texture points onto the sphere, the surface of the sphere doesn't change with changing resolutions. 


### Task 5: Environment-mapped Reflections
Our job in this task is to update <code class="language-plaintext highlighter-rouge">Mirror.frag</code>. We first calculate the outgoing eye-ray, $$w_o$$ by subtracting the camera's position, <code class="language-plaintext highlighter-rouge">u_cam_pos</code> from the fragment's position, <code class="language-plaintext highlighter-rouge">v_position</code>. Then, using the surface normal vector, we calculate the incoming $$w_i$$ by calculating $$w_o - 2 (w_o \cdot n) * n$$. Finally, we sample the <code class="language-plaintext highlighter-rouge">texture</code> for the incoming direction $$w_i$$ calculated.

Below are screenshots of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> with environment-mapped reflections.
<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/part5/mirror_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, mirror shader on sphere</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/part5/mirror_sphere_with_cloth.png" width="100%"/>
      <figcaption>../scene/sphere.json, mirror shader on sphere and cloth</figcaption>
    </td>
  </tr>
  <tr>
      <td colspan="2" align="center">
        <img src="../assets/hw4/part5/mirror_sphere_cloth.png" width="50%"/>
        <figcaption>../scene/sphere.json, mirror shader with cloth over sphere</figcaption>
      </td>
    </tr>
  </table>
</div>

Our custom shader is described in depth in [Part 6](/hw4.md#custom-shader-jerover-blue-so-true). 

## Part 6: Extra credit
For extra credit, we implemented [wind](/hw4.md#whoosh-its-windy-out-here), a [custom shader](/hw4.md#custom-shader-jerover-blue-so-true), and [collisions with cubes](/hw4.md#collisions-with-cubes).

### Whoosh! (It's windy out here)
We decided to add wind to our simulation by adding it into the `external_accelerations`, much like gravity. However unlike gravity, we wanted our wind to be variable at different timesteps in the simulation, as some static wind force would cause the cloth to settle into some equilibrium position and appear unnatural.

We tried a few approaches in order to simulate this behavior, the first of which was to try and simulate osciallation with sine and cosine functions. However, we found this to be too predictable and did not feel like natural wind behavior.

The next approach we used, which ended up being our final result, was empirically found and relied heavily on randomness. The idea is that the wind value is limited between 0 and a certain max value. If the current value for wind acceleration is near the mean value of the range, on the next iteration it would be encouraged to move towards the extremities. If the current value is near the extremities of the range, it would be encouraged to move towards the middle of the range. Of course, this is all probabilistic, with a certain degree of randomness injected into every calculation.

Below are screenshots of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/pinned2.json</code> while changing the wind value.
<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/ec/wind/positive_wind.png" width="100%"/>
      <figcaption>../scene/pinned2.json, positive wind value</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/ec/wind/negative_wind.png" width="100%"/>
      <figcaption>../scene/sphere.json, negative wind value</figcaption>
    </td>
  </tr>
  </table>
</div>

and you can see here that negative wind pushes the cloth towards the right while positive wind pushes it to the left.

We also include here a .gif of dynamically updating the wind values and its effect on a pinned cloth.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/ec/wind/windy.gif" width="50%"/>
        <figcaption>whoosh!</figcaption>
      </td>
    </tr>
  </table>
</div>

### Custom Shader: Jerover Blue, So True
We then implemented a custom shader in `Custom.frag` that was a hybrid between [mirror](/hw4.md#task-5-environment-mapped-reflections) and [phong](/hw4.md#task-2-blinn-phong-shading) shading. We added an extra interpolation between the color from mirror shading and interpolated that with the color [Jerover Blue](https://colornames.org/color/020f61), interpolating with an alpha value using [Schlick's approximation](https://en.wikipedia.org/wiki/Schlick%27s_approximation). Finally, in the `out_color`, we set $$\alpha$$ to be 0.5 to create a transparent output.

The resulting shader creates a blue-tinted transparent plastic material, shown by the images of <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code> below.
<div align="center">
<table style="width:100%">
  <colgroup>
      <col width="50%" />
      <col width="50%" />
  </colgroup>
  <tr>
    <td align="center">
      <img src="../assets/hw4/ec/shader/jeroverblue.png" width="100%"/>
      <figcaption>../scene/sphere.json, double reflected bridge</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/ec/shader/jeroverblue2.png" width="100%"/>
      <figcaption>../scene/sphere.json, mirror and phong combination</figcaption>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="../assets/hw4/ec/shader/missing_sphere.png" width="100%"/>
      <figcaption>../scene/sphere.json, where did the sphere go?</figcaption>
    </td>
    <td align="center">
      <img src="../assets/hw4/ec/shader/transparent.png" width="100%"/>
      <figcaption>../scene/sphere.json, the sphere is still here</figcaption>
    </td>
  </tr>
  </table>
</div>

The transparency that we mimicked in this shader almost makes it look like the sphere is missing when the cloth is put over it! However, when looking from the underside, we can see the sphere almost like a crystal orb.

Building on this, we also modified `Custom.vert` to displace the vertices of the sphere to simulate a bouncy/non-uniform sphere. We used a [random-number generator](https://stackoverflow.com/questions/4200224/random-noise-functions-for-glsl), making oscillating displacements using a combination of `sin` and `cos` to displace each vertex at each axis. We also used a few magic numbers in our displacement calculations, fit to our liking for bounciness.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/ec/shader/bouncy.gif" width="50%"/>
        <figcaption>bouncy wheeeee!</figcaption>
      </td>
    </tr>
  </table>
</div>


Here, we've combined our wind and custom fragment shader (not including the vertex shifts) in the following .gif of running <code class="language-plaintext highlighter-rouge">./clothsim -f ../scene/sphere.json</code>.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw4/ec/shade_and_wind.gif" width="50%"/>
        <figcaption>jerover whoosh!</figcaption>
      </td>
    </tr>
  </table>
</div>

### Collisions with Cubes
TODO

## Contributors
Edward Park, Ashley Chiu

TODO: We are best friends !

We really enjoyed working on this project :) We worked mainly together in person for Parts 1-5. Then spring break started, so we worked asynchronously, keeping each other up on what progress we made (specifically since we only had extra credit left). For extra credit, we split it up so Eddie worked on wind while Ashley worked on the custom shader, and then we worked together on collisions with cubes. In general, we collaborated well because we were able to work through a bulk of the project together in person, which mitigated any miscommuncations, and when we worked asynchronously, we were able to discuss and debug harder problems. 