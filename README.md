# cpu_rasterizer

A real-time 3D rendering pipeline written from scratch in JavaScript. No WebGL, no Three.js, no graphics API of any kind. Every triangle is transformed, clipped against the screen, shaded and depth-tested by hand, then written one pixel at a time into a raw byte buffer that gets blitted to a `<canvas>` with `putImageData`.

**[Live demo](https://apollo-2006.github.io/cpu_rasterizer/)**

The point is not performance. A GPU does this several thousand times faster and has since about 1999. The point is that after you write one, nothing in a graphics API is magic anymore: you know what a projection matrix is doing to your vertices, why the depth buffer stores what it stores, and what "backface culling" actually costs.

---

## What it does

Renders a procedurally generated torus, rotating on two axes, with flat directional shading and correct depth occlusion, at 400x300 internal resolution scaled up with nearest-neighbour filtering.

Three toggles expose the stages of the pipeline:

- **Barycentric wireframe** draws only pixels near a triangle edge, using the same coverage test that fills the solid version.
- **Backface culling** can be switched off to see the interior faces the cross-product test normally discards.
- **Depth buffer visualization** blits the depth buffer itself as greyscale instead of the color buffer.

---

## The pipeline

Each frame runs the full sequence. Nothing is cached between frames.

**1. Geometry.** `generateTorus(segments, rings, R, r)` walks a 2D parameter space and emits two triangles per quad, with consistent winding so the culling test downstream is meaningful.

**2. Model transform.** Rotation matrices around Z and X are composed with a translation and applied to every vertex. Row-vector convention, so the vertex is multiplied on the left and matrices compose left to right in application order.

**3. Backface culling.** The surface normal comes from the cross product of two triangle edges. If the dot product of that normal with the camera ray is positive, the face points away from the viewer and is discarded before it costs anything. This is roughly half the geometry on a closed mesh.

**4. Flat shading.** One dot product between the surface normal and a normalized light direction, clamped to a floor so nothing goes fully black. Per triangle, not per pixel, which is what makes it flat shading rather than Gouraud or Phong.

**5. Projection.** The perspective matrix maps view space into clip space and, critically, writes the view-space depth into `w`. The perspective divide by `w` is what makes distant geometry smaller.

**6. Rasterization.** For each triangle, compute a screen-space bounding box, then for every pixel inside it evaluate three barycentric weights. If all three are non-negative, the pixel is inside the triangle. The weights are divided by the signed area, which makes the test correct for either winding order without a special case.

**7. Depth test.** The interpolated depth is compared against the depth buffer, and the pixel is only written if it is closer.

**8. Blit.** `new ImageData(colorBuffer, w, h)` and one `putImageData` call. The color buffer is a `Uint8ClampedArray` of RGBA bytes written directly by index.

---

## The bug worth documenting

The first working version had a depth buffer that did nothing.

The perspective divide was implemented as `Vec3.div(v, v.w)`, and `Vec3.div` returned a new vector built from the constructor, whose `w` parameter defaults to `1`. So the divide silently discarded the only depth information the projection had produced. Every vertex afterwards reported `w = 1`, the depth stored per pixel was `1 / 1` for the entire frame, and the depth buffer held exactly two distinct values: `0` for background and `1` for everything drawn.

The result was not a crash. It was first-triangle-wins ordering that mostly looked fine, because backface culling already removes most hidden geometry on a convex object. A torus is not convex, so the near side of the ring should occlude the far side. Rendering a full rotation with and without the fix and diffing the framebuffers:

```
theta=0.00   wrong pixels:    60  (0.7%)
theta=0.70   wrong pixels:   262  (3.1%)
theta=1.40   wrong pixels:  1312  (23.8%)   <- ring edge-on
theta=1.75   wrong pixels:  1144  (20.5%)
theta=2.10   wrong pixels:   132  (1.7%)
theta=4.55   wrong pixels:  1188  (21.8%)
```

At typical angles the error is a couple of percent, which reads as flicker rather than obvious breakage. At the angles where the ring is edge-on and self-occluding, roughly a quarter of the model draws wrong.

Two things came out of fixing it.

The depth value stored is `1/w`, not the post-divide `z`. Only `1/w` interpolates linearly in screen space; interpolating post-divide `z` across a triangle gives a value that is wrong everywhere except the vertices. Since `1/w` gets larger as geometry gets closer, the buffer is cleared to `0` (infinitely far) and the test keeps the larger value.

`1/w` is now captured per vertex before the divide happens, and `Vec3` arithmetic carries `w` through instead of resetting it. Both, because either alone leaves the trap in place for the next person: the screen-space offset step also runs through `Vec3.add`, which would have discarded `w` a second time.

The depth visualization toggle exists because of this bug. A working depth buffer shows a smooth gradient across a curved surface; a broken one shows flat grey. It would have caught the whole thing in about four seconds.

---

## Running it

```bash
npm install
npm run dev        # dev server with HMR
npm run build      # production build into dist/
npm run deploy     # publish dist/ to GitHub Pages
```

Vite is configured with `base: '/cpu_rasterizer/'` for the Pages deployment. If you fork this under a different repo name, change that or asset paths will 404.

---

## Structure

```
src/
  App.jsx        Vec3, Mat4x4, Triangle, torus generator,
                 SoftwareRasterizer, and the render loop
  main.jsx       React entry point
  index.css      Tailwind
```

The math and the rasterizer are plain classes with no React dependency. React is the control panel and the canvas host, nothing more. The render loop runs on `requestAnimationFrame` inside a `useEffect` and is cancelled on unmount.

---

## Stack

JavaScript, React 19, Vite, Tailwind. Canvas 2D used only as a framebuffer target.
