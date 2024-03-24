---
layout: page
title: 'Homework 4: Clothsim'
has_right_toc: true
---
<p class="warning-message">
This assignment has not been completed yet.
</p>

## Overview
TODO

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
Our main goal in this part is completing the <code class="language-plaintext highlighter-rouge">Cloth::simulate</code> method that runs one time step of time length <code class="language-plaintext highlighter-rouge">delta_t</code> and applies all <code class="language-plaintext highlighter-rouge">accelerations</code> uniformly to all point masses in the cloth.

### Task 1: Compute total force acting on each point mass
First, we calculated all the external forces based on the <code class="language-plaintext highlighter-rouge">external_accelerations</code>, which just contains <code class="language-plaintext highlighter-rouge">gravity</code> as of now (see our [wind](/hw4.md#whoosh-wind) simulations for an updated <code class="language-plaintext highlighter-rouge">external_accelerations</code>). Since we're applying all accelerations uniformly, we can just sum them once, using Newton's 2nd Law. $$F = ma$$ (noting that the $$m$$, mass, is constant). Then, we set each point mass's <code class="language-plaintext highlighter-rouge">forces</code> to a <code class="language-plaintext highlighter-rouge">Vector3D</code> of our summed external forces.

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

We see that <code class="language-plaintext highlighter-rouge">density</code> operates almost inversely to <code class="language-plaintext highlighter-rouge">ks</code>. Namely, at the lowest <code class="language-plaintext highlighter-rouge">density = 1 g/cm^2</code>, we see the most rigid cloth, where there are less deformations in the cloth. Because at lower densities, the cloths will have lower mass, this means that the forces (that we accumulate) at each point masss will be less, and thus, means that there's less forces pulling the cloth down, causing less wrinkles. In contrast, at our highest <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>, there are larger external forces working on the point masses (due to Newton's 2nd Law). This means that the cloth weighs more, and thus, has many more wrinkles, which we can visibly see in the normal appearance with <code class="language-plaintext highlighter-rouge">density = 5,000 g/cm^2</code>.

When discussing how the cloth behaves as it moves from start to rest, we note that lower <code class="language-plaintext highlighter-rouge">density</code> values, the cloth weighs less, and as such, stays relatively flat as the forces acting against it aren't as strong. This differs from using higher <code class="language-plaintext highlighter-rouge">density</code> values as the forces acting on the cloth are now stronger, and thus, the cloth deforms more, which causes more waves and wrinkles as it swings down.

#### Changing <code class="language-plaintext highlighter-rouge">damping</code>
While maintaining the default <code class="language-plaintext highlighter-rouge">ks = 5000 N/m</code> and <code class="language-plaintext highlighter-rouge">density = 15 g/cm^2</code>, let's show ../scene/pinned2.json with <code class="language-plaintext highlighter-rouge">damping = 0%</code> and <code class="language-plaintext highlighter-rouge">damping = 1%</code>.

Here, we've included .gif depictions because what we believ to be more important is demonstrating the speed and flexibility at which the cloth moves. 

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

Finally, we compare what happens when we mess with  <code class="language-plaintext highlighter-rouge">damping</code>. At the lowest possible <code class="language-plaintext highlighter-rouge">damping = 0%</code> provided, we see that the cloth swings quiet quickly back and forth. In contrast, at the highest possible  <code class="language-plaintext highlighter-rouge">damping = 1%</code>, the cloth moves really quickly and seemingly stops at the rest position (that matches the rest position with default parameters). Since we know that damping messes with the velocity term in Verlet integration, with no damping, this means there is no loss of energy due to friction, heat loss, or any other force. Therefore, the cloth just swings back and forth, with wrinkling and no rigid structure because there is no loss of energy. However, with the highest damping, the cloth falls much slower and holds its structure a bit better. The expalanation for this is that the forces acting on it (namely, gravity) are now dampened, which means that the positions of the point masses don't move as quickly. This reflects a state in which there is a loss of energy, so not all the forces that are thrust upon the cloth actually convert into energy that moves the cloth--some is lost or dissipated.

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
Finally, we implemented our full self-collision method. For the passed in point mass, we check whether the normalized distance between it and all candidate point masses is less than $$2 * thickness$$, and also whether it's normalized distancen is greater than 0 (to prevent a point mass from colliding with itself). Then, we would calculate the final correction to be unit distance between the two point masses multipled by the correction to ensure that the pair would be $$2 * thickness$$ distance apart.

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

### Experimenting with <code class="language-plaintext highlighter-rouge">density</code> and <code class="language-plaintext highlighter-rouge">ks</code>
TODO

## Part 5: Shaders
TODO

## Part 6: Extra credit
TODO

### Whoosh! (wind)

## Contributors
Edward Park, Ashley Chiu

TODO: We are best friends !