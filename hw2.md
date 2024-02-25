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
        <img src="../assets/hw2/task1/task1_1.png" width="100%"/>
        <figcaption>Original control points</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1/task1_2.png" width="100%"/>
        <figcaption>Step 1</figcaption>
      </td>
    </tr>
      <tr>
      <td align="center">
        <img src="../assets/hw2/task1/task1_3.png" width="100%"/>
        <figcaption>Step 2</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1/task1_4.png" width="100%"/>
        <figcaption>Step 3</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task1/task1_5.png" width="100%"/>
        <figcaption>Step 4</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1/task1_6.png" width="100%"/>
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
        <img src="../assets/hw2/task1/task1_7.png" width="400px"/>
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
        <img src="../assets/hw2/task1/task1_8.png" width="400px"/>
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
        <img src="../assets/hw2/task1/task1_9.png" width="100%"/>
        <figcaption>Decreasing <code class="language-plaintext highlighter-rouge">t</code></figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task1/task1_10.png" width="100%"/>
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
        <img src="../assets/hw2/task2/task2.png" width="400px"/>
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
        <img src="../assets/hw2/task3/task3_1.png" width="100%"/>
        <figcaption>dae/teapot.dae with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task3/task3_2.png" width="100%"/>
        <figcaption>dae/teapot.dae with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 4: Edge flip

Utilizing this diagram, we mapped out the pointer changes for edge flips. 

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4/task4_diagram.png" width="600px"/>
        <figcaption>Demonstrating vertices, edges, halfedges, and faces before and after an edge flip.</figcaption>
      </td>
    </tr>
  </table>
</div>

Our main implementation strategy was first checking whether the edge to flip was a boundary, and if so, returning. Then, we stored all the current elements as diagrammed above (with the same naming conventions). Then, we <code class="language-plaintext highlighter-rouge">setNeighbors</code> for all the halfedges, ensuring that their <code class="language-plaintext highlighter-rouge">next</code>, <code class="language-plaintext highlighter-rouge">twin</code>, <code class="language-plaintext highlighter-rouge">vertex</code>, <code class="language-plaintext highlighter-rouge">edge</code>, and <code class="language-plaintext highlighter-rouge">face</code> matched that of the diagram we've shown above. Following that, we set each of the vertice's halfedges arbitrarily (so long as it was an outgoing halfedge). Then, we set each edge's halfedge and set each face's halfedge.

The main debugging trick we came up with was drawing out this diagram to ensure that we knew where everything was following the flip. This was also a great check to make sure we weren't deleting or adding any edges, halfedges, faces, or vertices. We were lucky to not have a tumultuous debugging journey.

One debugging error that we ran into was that certain mesh objects wouldn't change (for instance, the outer edges and halfedges didn't move annd the vertices stayed put) and we didn't originally update them. However, once we updated these values, it worked as expected.

Below are images of dae/teapot.dae before and after edge flips, with default flat and Phong shading.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4/task4_1.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task4/task4_2.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge flips with Phong shading</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4/task4_3.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task4/task4_4.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge flips with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 5: Edge split

Utilizing this diagram, we mapped out the pointer changes for edge splits. 

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_diagram.png" width="600px"/>
        <figcaption>Demonstrating vertices, edges, halfedges, and faces before and after an edge split.</figcaption>
      </td>
    </tr>
  </table>
</div>

Our main implementation strategy was first checking whether the edge to split was a boundary, and if so, we implemented boundary checking the [extra credit](/hw2.md#extra-credit-beep-beep) (so check that out!). Then, we stored all the current elements as diagrammed above (with the same naming conventions). We created all the necessary new elements (checking boundary conditions), specifically in the case with no boundary: 6 new halfedges, 3 new edges, 1 new vertex, and 2 new faces.

Our new vertex was set as the direct midpoint of the edge we were splitting (calculating the distance from the two vertices connecting that edge).

Then, we <code class="language-plaintext highlighter-rouge">setNeighbors</code> for all the halfedges, ensuring that their <code class="language-plaintext highlighter-rouge">next</code>, <code class="language-plaintext highlighter-rouge">twin</code>, <code class="language-plaintext highlighter-rouge">vertex</code>, <code class="language-plaintext highlighter-rouge">edge</code>, and <code class="language-plaintext highlighter-rouge">face</code> matched that of the diagram we've shown above. Following that, we set each of the vertice's halfedges arbitrarily (so long as it was an outgoing halfedge). Then, we set each edge's halfedge and set each face's halfedge.

The main debugging trick we came up with was drawing out this diagram to ensure that we knew where everything was following the split. This was also a great check to make sure we weren't deleting or adding any edges, halfedges, faces, or vertices. Following [Task 4](/hw2.md#task-4-edge-flip), we were a lot more careful in our diagram construction, and this meant we didn't actually have any difficult debugging.

Below are images of dae/teapot.dae before and after edge flips and splits, as captioned, with default flat and Phong shading.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task4/task4_1.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge splits with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task4/task4_2.png" width="100%"/>
        <figcaption>dae/teapot.dae, no edge splits with Phong shading</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_1.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5/task5_2.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits with Phong shading</figcaption>
      </td>
    </tr>
     <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_3.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits and flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5/task5_4.png" width="100%"/>
        <figcaption>dae/teapot.dae, edge splits and flips with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

Below, we constructed on the ending image and added more edge splits on the already split and flipped dae/teapot.dae to make an interesting pattern. Enjoy!

<div>
    <table>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_5.png" width="100%"/>
        <figcaption>dae/teapot.dae, even more edge splits and flips with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5/task5_6.png" width="100%"/>
        <figcaption>dae/teapot.dae, even more edge splits and flips with Phong shading</figcaption>
      </td>
    </tr>
</table>
</div>

### Extra credit (beep beep!)

Utilizing this diagram, we mapped out the pointer changes for edge splits on boundaries (if there is a left boundary). 

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_extracredit_diagram.png" width="600px"/>
        <figcaption>Demonstrating vertices, edges, halfedges, and faces before and after an edge split on a boundary.</figcaption>
      </td>
    </tr>
  </table>
</div>

Handling boundary splits was difficult in understanding how to navigate the halfedges that lined the the central two edges. However, when we realized that the halfedges didn't need to form triangles, this solved our problems, as we could have, for instance, halfedge <code class="language-plaintext highlighter-rouge">h_9</code>'s <code class="language-plaintext highlighter-rouge">next</code> point to halfedge <code class="language-plaintext highlighter-rouge">h_1</code> if we were not creating the <code class="language-plaintext highlighter-rouge">e_7</code> edge as diagrammed above.

We struggled in the beginning, first only creating one halfedge on the side that wasn't split. However, this caused problems in identifying <code class="language-plaintext highlighter-rouge">twin</code>s of halfedges and as such, we modified to this approach of allowing for a quadrilateral to show up in the mesh.

For implementation, we checked whether we would need to create a left boundary or a right boundary, and based on that, we changed the vertices, edges, halfedges, and faces for that. For instance, if we had a left boundary, this means that we wouldn't want to be constructing <code class="language-plaintext highlighter-rouge">e_5</code>, which meant that in our construction, we wouldn't need <code class="language-plaintext highlighter-rouge">h_7</code> or <code class="language-plaintext highlighter-rouge">h_8</code>, nor <code class="language-plaintext highlighter-rouge">f_2</code> (since we could use <code class="language-plaintext highlighter-rouge">f_0</code> for the entire left side). Then, we'd <code class="language-plaintext highlighter-rouge">setNeighbors</code> of halfedges and set halfedges of other mesh objects as coordinating to the diagram.

Doing this, we would need to create 4 new halfedges, 2 new edges, 1 new vertex, and 1 new face.

Here is a diagram of dae/beetle.dae before any boundary edge splitting. Notice that for the highlighted white edge, its <code class="language-plaintext highlighter-rouge">isBoundary()</code> is <code class="language-plaintext highlighter-rouge">1</code>, which means this is a boundary edge. This makes sense since we've hit the edge of a surface (the window!).

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_extracredit1.png" width="400px"/>
        <figcaption>dae/beetle.dae, highlighting a boundary edge</figcaption>
      </td>
    </tr>
  </table>
</div>

Here are images of dae/beetle.dae after splitting the edges around the nearest window in a pattern with both default flat and Phong shading. Beeep beep!

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task5/task5_extracredit2.png" width="100%"/>
        <figcaption>dae/beetle.dae, splitting a boundary edge with default flat shading</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task5/task5_extracredit3.png" width="100%"/>
        <figcaption>dae/beetle.dae, splitting a boundary edge with Phong shading</figcaption>
      </td>
    </tr>
  </table>
</div>

## Task 6: Loop subdivision for mesh upsampling

For our final task, we needed to implement loop subdivision. To do so, we had five main steps to go through: computing new positions for all existing vertices, (2) compute positions for new vertices, (3) split all edges in the mesh, (4) flip edges between an old and new vertex, and (5) copy over the vertex positions. The algorithm we utilized was a 4-1 subdivision, breaking each triangle down into four smaller ones.

### Compute new positions for existing vertices
Iterating through all vertices of the mesh, we found the halfedges associated with the vertices and calculated the sum of the positions of adjacent vertices, and depending on the degree, calculated a new position and stored it in the vertex's <code class="language-plaintext highlighter-rouge">newPosition</code> attribute. Namely, the new position would be
{% highlight js %}
(1 - n * u) * original_position + u * original_neighbor_position_sum
{% endhighlight %}

where 
- <code class="language-plaintext highlighter-rouge">n</code> represented the number of edges incident to the vertex (which we calculated by traversing halfedges and twins),
- <code class="language-plaintext highlighter-rouge">u</code> was <code class="language-plaintext highlighter-rouge">3/16</code> if <code class="language-plaintext highlighter-rouge">n = 3</code>, else <code class="language-plaintext highlighter-rouge">3/(8n)</code>,
- <code class="language-plaintext highlighter-rouge">original_position</code> was the original position of the vertex as named, and
- <code class="language-plaintext highlighter-rouge">original_neighbor_position_sum</code> was the sum of all the original positions of neighboring vertices (calcualted via traversing halfedges and twins).

### Compute positions for new vertices
Next, we needed to compute positiosn for new vertices that we were creating. Namely, for each pair of two triangles, we would be creating one new vertex. To do so, we traversed through the edges, and for each edge, we found the four surrounding edges, and surrounding four vertices, and calculated
{% highlight js %}
3/8 * (A + B) + 1/8 * (C + D)
{% endhighlight %}

where <code class="language-plaintext highlighter-rouge">A</code>, <code class="language-plaintext highlighter-rouge">B</code>, <code class="language-plaintext highlighter-rouge">C</code>, and <code class="language-plaintext highlighter-rouge">D</code> were all positions of the four vertices and specifically, <code class="language-plaintext highlighter-rouge">A</code> and <code class="language-plaintext highlighter-rouge">B</code> were the two vertices that connected the current edge. This would be stored into the current edge's <code class="language-plaintext highlighter-rouge">newPosition</code> attribute to create the new vertex. This is particularly useful for the next step because when we're splitting the edges, we can then use this <code class="language-plaintext highlighter-rouge">newPosition</code> for our newly created vertex.

### Split all edges
Now, we iterate through all the edges and split them (copying over the <code class="language-plaintext highlighter-rouge">newPosition</code> found in the edge from the [previous section](/hw2.md#compute-positions-for-new-vertices)) to be the position of the new vertex created from the split.

Something difficult in debugging here was ensuring that we weren't infinite looping due to the creation of new edges via the splits, so we would store the next edge we needed to traverse and create a temporary variable to store the current one, performing the split on the temporary pointer.

### Flip edges between new and old vertices
Next, we iterated through all the edges again, and now flipped them if it was connected to an old, existing vertex on one side and a new one on the other. 

### Copy vertex positions
Finally, for all vertices, we copied their <code class="language-plaintext highlighter-rouge">newPosition</code> into their <code class="language-plaintext highlighter-rouge">position</code> attribute since we knew that for both new and existing vertices, we wrote their new positions in the mesh in the <code class="language-plaintext highlighter-rouge">newPosition</code> attribute.

An error that we ran into was weird subdivisions following the first iteration and we realized that for both vertices and edges, we weren't resetting the <code class="language-plaintext highlighter-rouge">isNew</code> attribute correctly, and as such, we set these modifications to fix that, modifying [section one](/hw2.md#compute-new-positions-for-existing-vertices) so all existing vertices were marked as not new, [section two](/hw2.md#compute-positions-for-new-vertices) so existing edges were marked as not new, and [section three](/hw2.md#split-all-edges) so new vertices were marked new. Furthermore, we had to update our [Task 5](/hw2.md#task-5-edge-split) implementation to ensure that of the new edges we were creating, the ones that were created during the split were marked new while the ones that composed of the original edge in the middle was marked not new.

### Mesh behavior
After loop subdivision, we see that meshes with sharp corners (like dae/cube.dae) smooth out a great deal because we are increasing the number of triangles, which allows for more smooth curves to develop. However, this means that edges are lost. For instance, let's take a look at dae/cube.dae. Namely, we can see that if we angle it in one way, we can see it smooth out with more loop subidivison. 

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_1.png" width="100%"/>
        <figcaption>dae/cube.dae, loaded </figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_2.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 1</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_3.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 2</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_4.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 3</figcaption>
      </td>
    </tr>
  </table>
</div>

We can reduce this effect by pre-splitting some edges. Below, we've pre-split some edges for the closest vertex (to preserve that vertex's sharpness), and we can see that the sharpness there remains while the rest of the image smooths out.
<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_13.png" width="100%"/>
        <figcaption>dae/cube.dae, loaded with pre-split edges</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_14.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 1</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_15.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 2</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_16.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 3</figcaption>
      </td>
    </tr>
  </table>
</div>

From a front-on angle, we can really see the impact of the pre-splitting.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_17.png" width="400px"/>
        <figcaption>dae/cube.dae, highlighting vertex sharpness</figcaption>
      </td>
    </tr>
  </table>
</div>

### Loop division asymmetry

The following screenshots show several iterations of loop subdivision on dae/cube.dae. We notice that the cube becomes slightly asymmetric after repeated subdivisions. We believe this occurs because the original mesh itself was not symmetric: the asymmetry arose because the triangle orientations were not shared across all the faces, and this caused subdivisions to pull more in one direction than the other.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_1.png" width="100%"/>
        <figcaption>dae/cube.dae, loaded</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_2.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 1</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_3.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 2</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_4.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 3</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_5.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 4</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_6.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5</figcaption>
      </td>
    </tr>
  </table>
</div>

In the following screenshots, we've pre-processed dae/cube.dae, splitting the diagonal edge on each of the six faces. We believe that this created a more symmetrical cube after iterations of loop subdivision because each face was now symmetrical with four triangles and this meant that each vertex had the same degree (so there was no unequal pull in directions when splitting and calculating new vertex positions).

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_7.png" width="100%"/>
        <figcaption>dae/cube.dae, loaded, pre-processed with one split on each diagonal edge on each face</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_8.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 1, after pre-processing</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_9.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 2, after pre-processing</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_10.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 3, after pre-processing</figcaption>
      </td>
    </tr>
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_11.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 4, after pre-processing</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_12.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5, after pre-processing</figcaption>
      </td>
    </tr>
  </table>
</div>

We can really see the difference in asymmetry when we compare the dae/cube.dae images after loop subdivision at step 5 with no pre-processing (on the left) and with pre-processing (on the right). The one on the right is much more symmetrical while the one on the left seems to have a weird bump in the back right and bumpiness as well at the bottom.

<div align="center">
  <table style="width:100%">
    <tr>
      <td align="center">
        <img src="../assets/hw2/task6/task6_6.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5, no pre-processing</figcaption>
      </td>
      <td align="center">
        <img src="../assets/hw2/task6/task6_12.png" width="100%"/>
        <figcaption>dae/cube.dae, loop subdivision step 5, after pre-processing</figcaption>
      </td>
    </tr>
  </table>
</div>

## Contributors
Ashley Chiu, Angel Aldaco
