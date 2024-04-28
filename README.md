# ProcessingRayTracer

Processing projects implementing ray tracing, implicit surface generation, and mesh manipulation.

This repo holds all my project implementations for Computer Graphics (CS-6491), Spring 2024, GA Tech.

## Run

* [Download Processing](https://processing.org/download)
* Open one of the project folders and hit the run button
* Use keys to load scenes (details below)

## Ray Tracing

### Ray Tracing: Results

The scenes for this project are given in pairs.
Pressing keys `1` to `9` causes the left (a) version of the scene to be rendered.
Pressing `shift` `1` to `9` (characters `!@#$%^&*(`) causes the right (b) scene to be rendered.
Many (but not all) of the (b) scenes shoot more than one ray per pixel.

Below are the the generated images:

| (a) | (b) |
|-----------|-----------|
| ![s01a.jpg](renders/ray_tracer/s01a.jpg) | ![s01b.jpg](renders/ray_tracer/s01b.jpg) |
| ![s02a.jpg](renders/ray_tracer/s02a.jpg) | ![s02b.jpg](renders/ray_tracer/s02b.jpg) |
| ![s03a.jpg](renders/ray_tracer/s03a.jpg) | ![s03b.jpg](renders/ray_tracer/s03b.jpg) |
| ![s04a.jpg](renders/ray_tracer/s04a.jpg) | ![s04b.jpg](renders/ray_tracer/s04b.jpg) |
| ![s05a.jpg](renders/ray_tracer/s05a.jpg) | ![s05b.jpg](renders/ray_tracer/s05b.jpg) |
| ![s06a.jpg](renders/ray_tracer/s06a.jpg) | ![s06b.jpg](renders/ray_tracer/s06b.jpg) |
| ![s07a.jpg](renders/ray_tracer/s07a.jpg) | ![s07b.jpg](renders/ray_tracer/s07b.jpg) |
| ![s08a.jpg](renders/ray_tracer/s08a.jpg) | ![s08b.jpg](renders/ray_tracer/s08b.jpg) |
| ![s09a.jpg](renders/ray_tracer/s09a.jpg) | ![s09b.jpg](renders/ray_tracer/s09b.jpg) |

Pressing `0` renders the custom scene from the `implicit_surfaces` project, rendered as a low-poly, single-colored instance with soft-shadows, DOF, and glossy reflections:

![](renders/ray_tracer/implicit_surface_scene.png)

### Ray Tracing: Scene description language

Each scene is described by a text file with `.cli` suffix (Command Language Interpreter), with each nonempty line describing a scene _command_.

Each project revoled around implementating new scene description commands.
Below are the commands, with explanations adapted from the assignments for clarity.

* `background  r g b`:
Sets the background color.
If a ray misses all the objects in the scene, the corresponding pixel is set to this color.
* `fov  angle`:
Specifies the field of view (in degrees) of the virtual camera.
* `light  x y z  r g b`:
Create a point light source at position `(x,y,z)` and its color `(r, g, b)`.
Any number of light sources is allowed.
* `surface  dr dg db`:
Specifies a new surface material with diffuse color `(dr, dg, db)`.
* `begin`:
Indicates the start of a triangle.
* `vertex  x y z`:
Specifies one vertex of a triangle.
Three such vertex commands in a row define a triangle.
* `end`:
Indicates the end of a triangle.
* `render`:
Ray-traces the scene and displays the image in the drawing window.
* `translate  tx ty tz`:
Create a translation matrix $T$ and multiply it on the right-hand side of the matrix stack, $C_{\text{new}} = C * T$.
* `scale  sx sy sz`:
Create a scale matrix $S$ and multiply it on the right-hand side of the matrix stack, $C_{\text{new}} = C * S$.
* `rotatex  angle`, `rotatey  angle`, `rotatez  angle`, `rotate x_angle y_angle z_angle`:
Create a rotation matrix $R$ and multiply it on the right-hand side of the matrix stack, $C_{\text{new}} = C * R$.
Angles of rotation are specified in degrees.
* `push`:
Duplicate the matrix on top of the stack, and push this duplicate onto the stack.
This new top of the stack is the current transformation matrix $C$.
When the program starts with a single identity matrix $I$ on top of the stack.
* `pop`:
Pop off the top matrix from the stack and discard it.
If the stack only has one element when, an error is printed.
* `vertex  x y z`:
All triangles in the scene are affected by the current transformation matrix $C$ (the top matrix on the stack).
* `box  xmin ymin zmin  xmax ymax zmax`:
Create an axis-aligned box primitive, and put it on the list of scene objects.
Used for acceleration data structure.
Just like triangles, boxes are modified by the current transformation matrix when the box is created.
(Rotating an axis-aligned box will have unexpected results.)
* `named_object  <name>`:
Remove the most recently created object from the list of scene objects, convert this object to a named object, and place it in the named objects collection.
A named object can be a triangle, an axis aligned box, or an acceleration data structure that contains multiple objects.
* `instance <name>`:
Create an instance of a named object and add that object to the scene objects list.
* `begin_accel`:
Begin creating a list of objects separate from the list of scene objects.
* `end_accel`:
Finish putting objects into the separate acceleration list, place this list of objects into a bounding volume hierarchy, and ddd this new acceleration object into the scene.
* `read  filename`:
Begin reading commands from another file (e.g. `read bun500.cli`).
* `sphere  radius  x y z`:
Create a sphere with a given radius and center location.
* `rays_per_pixel  num`:
Specify how many rays per pixel to shoot.
One ray per pixel by default.
When `num > 1`, sub-pixel rays are created in random positions within each pixel.
The colors of these rays are averaged together to determine the final color of the pixel.
* `moving_object  dx dy dz`:
Change the last object that was defined into a moving object.
The displacement `(dx, dy, dz)` specifies the amount of translation this object undergoes during one frame.
Rays shot at such an object are assigned a random time in [0, 1].
The moving object is translated by this random time amount times the displacement.
When many `rays_per_pixel >> 1`, this gives the appearance of motion blur.
* `disk_light  x y z  radius  dx dy dz  r g b`:
Create a disk light source, with a given center `(x, y, z)`, radius, light direction `(dx, dy, dz)`, and light color `(r, g, b)`.
Shadow rays are shot to random locations on this disk.
When many `rays_per_pixel >> 1`, this creates soft shadows.
* `lens  radius  dist`:
When `radius > 0`, creates depth-of-field (DOF) effects by shooting rays from a lens with eye rays originating from a random point on the lens.
The lens is a disk that is centered at the origin of the given radius, perpendicular to the z-axis.
`dist` is the distance to the focal plane, perpendicular to the z-axis.
All rays for a given pixel, no matter where they originate on the lens, pass through the same point on the focal plane.
* `glossy  dr dg db  sr sg sb  spec_pow  k_refl  gloss_radius`:
Create a shiny surface material.
`(dr dg db)` are the diffuse color coefficients, just as in the `surface` command.
`(sr sg sb)` are the specular color coefficients, and only affect the colors of the specular highlights.
`spec_pow` affects the apparent roughness of the surface, with higher values giving tighter highlights.
`k_refl > 0` indicates the surface is shiny enough to shoot reflected rays and indicates how much they contribute to the surface color.
When `gloss_radius == 0`, the direction of the reflected rays travels in the perfect mirror direction.
When `gloss_radius > 0`, a "fuzz factor" is added to the perfect mirror direction, taken from a random vector inside a sphere of the given radius.
When multiple rays per pixel are used, this gives the impression of glossy reflections.

## Implicit Surfaces

The following are the scene keys, descriptions and renders for this project:

* **1:** A single implicit sphere.
* **!:** A flattened sphere that looks like a flying saucer.

||||
|-----------|-----------|-----------|
| ![s01a.jpg](renders/implicit_surfaces/s01a.jpg) | ![s01b.jpg](renders/implicit_surfaces/s01b.jpg) | ![s01c.jpg](renders/implicit_surfaces/s01c.jpg) |
| | ![s01d.jpg](renders/implicit_surfaces/s01d.jpg) | |

|**2:** Pairs of blobby spheres that are partially blended together.| **@:** A bunch of 10 randomly placed blobby spheres that are colored randomly.|
|-----------|-----------|
| ![s02a.jpg](renders/implicit_surfaces/s02a.jpg) | ![s02b.jpg](renders/implicit_surfaces/s02b.jpg) |
| ![s02c.jpg](renders/implicit_surfaces/s02c.jpg) | ![s02d.jpg](renders/implicit_surfaces/s02d.jpg) |

|**3:** Create and draw an implicit line segment.|**#:** Four blobby implicit line segments that blend together to form a blobby square.|
|-----------|-----------|
| ![s03a.jpg](renders/implicit_surfaces/s03a.jpg) | ![s03b.jpg](renders/implicit_surfaces/s03b.jpg) |

|**4:** An implicit torus.|**$:** Three blobby tori that are stuck to each other.|
|-----------|-----------|
| ![s04a.jpg](renders/implicit_surfaces/s04a.jpg) | ![s04b.jpg](renders/implicit_surfaces/s04b.jpg) |

|**5:** Create one thin implicit line segment that has its long axis parallel to the x-axis.|**%:** Take the thin line segment from Key 5 and warp its shape using a twist deformation.|
|-----------|-----------|
| ![s05a.jpg](renders/implicit_surfaces/s05a.jpg) | ![s05b.jpg](renders/implicit_surfaces/s05b.jpg) |

|**6:** Implement the taper deformation, and use it to create a tapered line segment.|**^:** Use both taper and twist together to deform a line segment.|
|-|-|
| ![s06a.jpg](renders/implicit_surfaces/s06a.jpg) | ![s06b.jpg](renders/implicit_surfaces/s06b.jpg) |

|**7:** Perform boolean intersection between two spheres to make a saucer shape that has sharp edges.|**&:** Perform a boolean subtraction that punches a hole through a sphere.|
|-----------|-----------|
| ![s07a.jpg](renders/implicit_surfaces/s07a.jpg) | ![s07b.jpg](renders/implicit_surfaces/s07b.jpg) |

Allow interactive morphing between a sphere and three intersecting line segments:

|||||
|-----------|-----------|-----------|-----------|
| ![s08a.jpg](renders/implicit_surfaces/s08a.jpg) | ![s08b.jpg](renders/implicit_surfaces/s08b.jpg) | ![s08c.jpg](renders/implicit_surfaces/s08c.jpg) | ![s08d.jpg](renders/implicit_surfaces/s08d.jpg) |

* **0:** My custom implicit surface scene incorporating all the above elements:

![](renders/implicit_surfaces/tree.png)

## Mesh Manipulation

### Mesh Manipulation: Key commands

* 1-7: Read in a mesh file (octahedron, cube, icosahedron, dodecahedron, star, torus, S).
* f: Toggle between per-face and per-vertex normals.
* w: Toggle between white and randomly colored faces.
* e: Toggle between not showing and showing the mesh edges in black.
* v: Toggle visualization of a directed edge on / off
* n: Change the current edge using the "next" operator.
* p: Change the current edge using the "previous" operator.
* o: Change the current edge using the "opposite" operator.
* s: Change the current edge using the "swing" operator.
* d: Create and display the dual of the mesh (unlimited number of times).
* g: Perform geodesic (midpoint) subdivision of the mesh and project vertices to sphere (unlimited number of times).
* c: Perform Catmull-Clark subdivision (unlimited number of times).
* r: Add random noise to the mesh by moving the vertices in the normal direction (both in and out).
* l (lower case "L"): Perform Laplacian smoothing (40 iterations).
* t: Perform Taubin smoothing (40 iterations).

### Mesh Manipulation: Results

**Adjacency operators:** In the top row, a directed edge is illustrated with three small spheres. The remaining four images show where this corner would be moved by each of the following operators: next, previous, opposite, and swing, respectively.

| Directed Edge | Next | Previous | Opposite | Swing |
|---------------|------|----------|----------|-------|
| ![](renders/mesh_manipulation/01_show_directed_edge.png) | ![](renders/mesh_manipulation/02_next.png) | ![](renders/mesh_manipulation/03_prev.png) | ![](renders/mesh_manipulation/04_opposite.png) | ![](renders/mesh_manipulation/05_swing.png) |

**Shading:** These next images show each model using flat shading, flat shading with edges shown, colored faces, and with per-vertex normals.

| Octahedron | Octahedron (Edges) | Octahedron (Colored) | Octahedron (Normals) |
|------------|---------------------|-----------------------|-----------------------|
| ![](renders/mesh_manipulation/10a_oct.png) | ![](renders/mesh_manipulation/10b_oct.png) | ![](renders/mesh_manipulation/10c_oct.png) | ![](renders/mesh_manipulation/10d_oct.png) |

| Cube | Cube (Edges) | Cube (Colored) | Cube (Normals) |
|------|--------------|----------------|----------------|
| ![](renders/mesh_manipulation/11a_cube.png) | ![](renders/mesh_manipulation/11b_cube.png) | ![](renders/mesh_manipulation/11c_cube.png) | ![](renders/mesh_manipulation/11d_cube.png) |

| Icosahedron | Icosahedron (Edges) | Icosahedron (Colored) | Icosahedron (Normals) |
|--------------|----------------------|------------------------|------------------------|
| ![](renders/mesh_manipulation/12a_icos.png) | ![](renders/mesh_manipulation/12b_icos.png) | ![](renders/mesh_manipulation/12c_icos.png) | ![](renders/mesh_manipulation/12d_icos.png) |

| Star | Star (Edges) | Star (Colored) | Star (Normals) |
|------|--------------|----------------|----------------|
| ![](renders/mesh_manipulation/13a_star.png) | ![](renders/mesh_manipulation/13b_star.png) | ![](renders/mesh_manipulation/13c_star.png) | ![](renders/mesh_manipulation/13d_star.png) |

**Dual meshes:** Each of the next rows shows an original object on the left and its dual on the right.

| Icosahedron | Icosahedron Dual |
|-------------|------------------|
| ![](renders/mesh_manipulation/20_icos.png) | ![](renders/mesh_manipulation/21_icos_dual.png) |

| Star | Star Dual |
|------|-----------|
| ![](renders/mesh_manipulation/22_star.png) | ![](renders/mesh_manipulation/23_star_dual.png) |

| Torus | Torus Dual |
|-------|------------|
| ![](renders/mesh_manipulation/24_torus.png) | ![](renders/mesh_manipulation/25_torus_dual.png) |

| S | S Dual |
|---|--------|
| ![](renders/mesh_manipulation/26_s.png) | ![](renders/mesh_manipulation/27_s_dual.png) |

**Geodesic (midpoint) subdivision:** Each row of the next images shows an original model, the model with one step of geodesic subdivision, two steps of geodesic subdivision, and three steps of geodesic subdivision. The two models at the left are the octahedron and the icosahedron.

| Octahedron | Octahedron (1 Step) | Octahedron (2 Steps) | Octahedron (3 Steps) |
|------------|----------------------|----------------------|----------------------|
| ![](renders/mesh_manipulation/30_oct.png) | ![](renders/mesh_manipulation/31_oct_geo1.png) | ![](renders/mesh_manipulation/32_oct_geo2.png) | ![](renders/mesh_manipulation/33_oct_geo3.png) |

| Icosahedron | Icosahedron (1 Step) | Icosahedron (2 Steps) | Icosahedron (3 Steps) |
|-------------|-----------------------|-----------------------|-----------------------|
| ![](renders/mesh_manipulation/34_icos.png) | ![](renders/mesh_manipulation/35_icos_geo1.png) | ![](renders/mesh_manipulation/36_icos_geo2.png) | ![](renders/mesh_manipulation/37_icos_geo3.png) |

**Catmull-Clark subdivision:** Each row of the next images shows an original model, followed by the model with 1, 2, and 4 steps of Catmull-Clark subdivision.

| Octahedron | Octahedron (1 Step) | Octahedron (2 Steps) | Octahedron (4 Steps) |
|------------|----------------------|----------------------|----------------------|
| ![](renders/mesh_manipulation/40a_oct.png) | ![](renders/mesh_manipulation/40b_oct.png) | ![](renders/mesh_manipulation/40c_oct.png) | ![](renders/mesh_manipulation/40d_oct.png) |

| Cube | Cube (1 Step) | Cube (2 Steps) | Cube (4 Steps) |
|------|----------------|----------------|----------------|
| ![](renders/mesh_manipulation/41a_cube.png) | ![](renders/mesh_manipulation/41b_cube.png) | ![](renders/mesh_manipulation/41c_cube.png) | ![](renders/mesh_manipulation/41d_cube.png) |

| Icosahedron | Icosahedron (1 Step) | Icosahedron (2 Steps) | Icosahedron (4 Steps) |
|-------------|-----------------------|-----------------------|-----------------------|
| ![](renders/mesh_manipulation/42a_icos.png) | ![](renders/mesh_manipulation/42b_icos.png) | ![](renders/mesh_manipulation/42c_icos.png) | ![](renders/mesh_manipulation/42d_icos.png) |

| Dodecahedron | Dodecahedron (1 Step) | Dodecahedron (2 Steps) | Dodecahedron (4 Steps) |
|--------------|------------------------|------------------------|------------------------|
| ![](renders/mesh_manipulation/43a_dodec.png) | ![](renders/mesh_manipulation/43b_dodec.png) | ![](renders/mesh_manipulation/43c_dodec.png) | ![](renders/mesh_manipulation/43d_dodec.png) |

| Star | Star (1 Step) | Star (2 Steps) | Star (4 Steps) |
|------|---------------|----------------|----------------|
| ![](renders/mesh_manipulation/44a_star.png) | ![](renders/mesh_manipulation/44b_star.png) | ![](renders/mesh_manipulation/44c_star.png) | ![](renders/mesh_manipulation/44d_star.png) |

| Torus | Torus (1 Step) | Torus (2 Steps) | Torus (4 Steps) |
|-------|----------------|----------------|-----------------|
| ![](renders/mesh_manipulation/45a_torus.png) | ![](renders/mesh_manipulation/45b_torus.png) | ![](renders/mesh_manipulation/45c_torus.png) | ![](renders/mesh_manipulation/45d_torus.png) |

| S | S (1 Step) | S (2 Steps) | S (4 Steps) |
|---|------------|------------|------------|
| ![](renders/mesh_manipulation/46a_s.png) | ![](renders/mesh_manipulation/46b_s.png) | ![](renders/mesh_manipulation/46c_s.png) | ![](renders/mesh_manipulation/46d_s.png) |

**Noise and smoothing:** Each of the next two rows shows a subdivided mesh, noise added, noise and Laplacian smoothing, noise and Taubin smoothing.
*Laplacian smoothing: lambda = 0.6. Taubin smoothing: lambda1 = 0.6307, lambda2 = -0.67315.*

Geodesic subdivision on icosahedron:

| Original | With Noise | With Noise and Laplacian Smoothing | With Noise and Taubin Smoothing |
|----------|------------|------------------------------------|----------------------------------|
| ![](renders/mesh_manipulation/50a_icos.png) | ![](renders/mesh_manipulation/50b_icos.png) | ![](renders/mesh_manipulation/50c_icos.png) | ![](renders/mesh_manipulation/50d_icos.png) |

Catmull-Clark subdivision on star:

| Original | With Noise | With Noise and Laplacian Smoothing | With Noise and Taubin Smoothing |
|----------|------------|------------------------------------|----------------------------------|
| ![](renders/mesh_manipulation/51a_star.png) | ![](renders/mesh_manipulation/51b_star.png) | ![](renders/mesh_manipulation/51c_star.png) | ![](renders/mesh_manipulation/51d_star.png) |


**Smooth shaded models:** These next three images show smooth shaded versions of models that were subdivided three times using Catmull-Clark.
The models are the icosahedron, star, and 'S'.

| Icosahedron | Star | S |
|-------------|------|---|
| ![](renders/mesh_manipulation/60_icos.png) | ![](renders/mesh_manipulation/61_star.png) | ![](renders/mesh_manipulation/62_s.png) |
