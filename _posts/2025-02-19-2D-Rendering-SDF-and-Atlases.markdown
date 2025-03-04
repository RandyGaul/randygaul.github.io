---
layout: post
title:  "2D Rendering with SDF's and Atlases"
date:   2025-03-04
categories: graphics
---

# Sprite Packing

In 2D game development, texture atlases are commonly used to reduce draw calls and optimize GPU performance. Traditionally, atlases are generated as a build-time preprocessing step, avoiding runtime overhead.

However, this approach complicates the artist workflow. Artists must manually sort images into atlases, slowing iteration and increasing risk when updating assets. If they could instead work with individual image files, iteration would be faster and more flexible.

With modern hardware—more RAM and widespread SSD adoption—we can shift atlas generation to runtime without noticeable performance impact. A sprite or shape-drawing API only needs transform data and a way to fetch pixels. Atlases can be compiled on demand, grouping assets dynamically based on actual draw usage. Unused assets can also be ejected from atlases as needed, optimizing memory without rigid preprocessing constraints.

This is a way to improve asset handling, balancing performance and workflow efficiency for artists and developers alike.

# Run-Time Atlas API

```cpp
Sprite sprite = make_sprite("path");
sprite.draw(at);
```

Pros
- Dead-simple drawing API
- No preprocessing or build step complications (no abstraction leaks)
- Things are always atlased together when drawn together

Cons
- More RAM consumption to hold assets in memory for adjusting atlases as-needed
- Minor perf hit for bookkeeping

`sprite.draw` can simply push plain-old-data (structs) into a buffer. The buffer can be processed as-needed by the sprite API to produce draw calls. The sprite API can, internally, figure out how to deal with texture atlases.

# Text

A full-featured rendering API in 2D would have three key drawable types: sprites, shapes, and text. We've already covered sprites so far. Text can be rendered in the same way sprites are rendered, by pushing sprites into the sprite API for rendering. A function such as `draw_text("hello")` works well. Each character can be decoded from a UTF8 input string one at a time. The character can map to a glyph within a font, where each glyph has a pre-rasterized image from the font file. `stb_truetype.h` is a good library to perform this font loading and rasterization step.

A great side-effect of our runtime sprite API is we can inject new sprites whenever we like without worrying about atlases. This includes applying prerendered effects for text such as blurring. This means only glyphs that are actually rendered need be uploaded to the GPU, and only glyphs that are blurred need be uploaded to the GPU. This creates a highly flexible system with very low VRAM requirements. Other more rigid text systems typically prerender glyphs onto atlases on-disk, and require some input to decide which glyphs the game will actually use at run-time. This is essentially an abstraction leak that leaks all the way into the asset pipeline.

# Shapes

Shapes can be rendered alongside the sprite/text API. A traditional 2D shape rendering approach could achieve antialiasing by generating skirt geometry around the borders of shapes, creating slightly opaque feathered edges. The skirts themselves must be calculated carefully, requiring custom code for each new type of shape to be rendered. This gets especially tricky for shapes like polylines. It gets especially complex when trying to round shape corners. Now imagine throwing in stroked shapes (drawing just the border with an empty interior). You're now in for an explosion of explicit code cases to write out by hand.

Example of drawing rounded, annular (stroked) shapes with and without antialias:

<div style="text-align: center;">
  <img src="/assets/aa_boxes.gif" alt="aa_boxes">
</div>

Instead, a simpler way to render shapes that naturally lends itself to both rounding and antialiasing is the concept of signed-distance functions, focusing on finding the isosurface of a shape. Take for example a circle. We can, for each pixel, calculate the distance of the pixel to the circle's surface:

```c
float sdf_circle(vec2 center, float radius, vec2 p)
{
	return distance(center, p) - radius;
}
```

It turns out Inigo Quilez has created a [nice collection of SDF functions for implicitly defined 2D shapes](https://iquilezles.org/articles/distfunctions2d/). Fragment shaders are good at determining the distance of each pixel to a particular shape. Given the distance you decide if a fragment is inside the shape (negative distance), outside the shape (positive distance), or close to the border (within some absolute tolerance). However, the shadertoy examples provided by Inigo run full-screen. To make practical use of this technique it's important to try and reduce pixel overdraw.

For 2D games pixel overdraw can become a serious bottleneck. This happens when the GPU is saturated with too many writes to the same pixel, causing performance loss. By wrapping each shape in a tightly bound quad pixel overdraw can be largely mitigated. The data we send to the GPU can look something like this design sketch:

Vertex attributes:
- Point for the quad (in world space)
- Homogeous point for the quad (transformed by the camera)
- Point for the circle (in world space)

Hardware interpolation between the vertex and fragment shaders should be used for these attributes. The homogenous point would be the output of the vertex shader for actual rasterization of the quad wrapping the shape. The other two attributes are fed into the SDF to calculate a distance.

To perform final rendering in the fragment shader a simple ternary works great to fill in the shape.

```glsl
float d = distance(center, radius, p);
out_color = d > 0.0 ? vec4(0) : vec4(1);
```

This ternary will not produce branching or cause any significant performance overhead. Hardware acceleration will be used to select the correct result.

To round the shape simply adjust the surface of the shape by a constant. This inflates the shape. Examples can be seen over at [Inigo's work](https://iquilezles.org/articles/distfunctions2d/). The same goes for antialiasing -- simply blur between boundaries over the region of distance near zero. `smoothstep` works great for this. Here are some examples of shape inflating:

<div style="text-align: center;">
  <img src="/assets/chubiness.gif" alt="chubiness">
</div>

<div style="text-align: center;">
  <img src="/assets/chubiness_triangle.gif" alt="chubiness_triangle">
</div>

Here's an example of drawing a blurred shape:

<div style="text-align: center;">
  <img src="/assets/aa_scale.gif" alt="aa_scale">
</div>

Here's an example of polygon rendering (up to 8 vertices):

<div style="text-align: center;">
  <img src="/assets/polygon_cf2.gif" alt="polygon_cf2">
</div>

Using the SDF technique all shape variety becomes isolated down to the SDF itself. Most other aspects of rendering shapes becomes agnostic to the shape type itself. This helps scale up the number of shapes to be supported without incurring too much of a development or maintenance cost. Here's a quick summary of how all these pieces fit together:

1. Wrap the shape in a tight-fitting quad
2. Attach attributes to each quad's vertices for the shape's information, as the fragment shader needs to know the full geometry of the shape in question
3. Send off draw calls
4. Evaluate the SDF for each shape
5. Apply shape rounding, antialiasing, or stroke effects

Steps 1, 2, and 3 require different code paths for different shapes. However, these steps are much simpler to implement compared to generating traditional feather geometry or traditional shape rounding/stroke geometry.

# Batching

One final consideration is how to fit different kinds of shapes into a single draw call, alongside text and sprites. Wouldn't it be cool to draw your whole game in just a couple of draw calls, including beautifully rendered antialiased shapes, sprites, and text all at once?

If we expand the attributes to include up to 8 different points all kinds of shapes can be represented. Triangles, line segments, circles, capsules, and polygons (up to 8 vertices) can all be rendered "together". I say "together" since in practice each shape type will get rendered serially, as the GPU will have to branch on shape types. However, in practice most games generally render just a couple major shape types, and lots of them. The low batch counts and overall performance will still be highly competitive.

The branching happens by passing in the type of shape as another vertex attribute. A simple if-else chain can be figure out what shape each vertex belongs to and map to the appropriate SDF.

```glsl
bool is_sprite  = v_type >= (0.0/255.0) && v_type < (0.5/255.0);
bool is_text    = v_type >  (0.5/255.0) && v_type < (1.5/255.0);
bool is_box     = v_type >  (1.5/255.0) && v_type < (2.5/255.0);
bool is_seg     = v_type >  (2.5/255.0) && v_type < (3.5/255.0);
bool is_tri     = v_type >  (3.5/255.0) && v_type < (4.5/255.0);
bool is_tri_sdf = v_type >  (4.5/255.0) && v_type < (5.5/255.0);
bool is_poly    = v_type >  (5.5/255.0) && v_type < (6.5/255.0);
```

In the above example 0 maps to sprite, 1 maps to text, 2 maps to box, and so on. Talking about performance, it's best to try and sort the internals of your draw call based on shape type as much as possible. The main performance hit from branching here comes from warp divergence. A GPU warp (or wavefront on AMD) is a group of parallel threads to compute the same instruction many time simultaneously. It's like a form of very wide SIMD. There's a tradeoff here of lower CPU driver overhead and stronger throughput (less and bigger draw calls). Let's outline a full pro/con list in terms of performance.

Pros
- Heavily reduced implementation complexity
- Fewer draw calls
- Strong memory coherence

Cons
- Potential warp divergence
- Attribute bloat
- Potential pixel overdraw

I've gotten away with ~144 bytes per-vertex for a full-featured 2D renderer. This is really quite reasonable, and can be paired down by stripping some features (polygon SDF rendering being the biggest culprit). As for warp divergence, in practice this really hasn't been an issue for any game I've seen so far. Games tend to draw a lot of the same shape type, such as many sprites, or many circles/lines. This lends itself to very high performance.

# Polylines

One difficulty I encountered with rendering polylines is coming up with a good triangulation that doesn't produce pixel gaps between segments. The core issue with polyline rendering is each segment in the chain of segments must be rendered with perfectly adjacent triangles. Vertex calculations must be done in a deterministic that doesn't produce slightly different results between segments. Not only this, but the SDF itself must be queried from one segment to another with the exact same inputs on the fragment shader. Failure to follow these rules results in missing and popping pixels along the seams between lines.

The strategy I went with produces a quad per segment, and potentially a wedge for narrow corners. Side note: hopefully you don't see the bugs in certain cases in this gif:

<div style="text-align: center;">
  <img src="/assets/aa_lines_geom.gif" alt="aa_lines_geom">
</div>

Bugs aside, as long as adjacent vertices from one segment to another are identical you can achieve a perfect render. For the SDF's themselves I pass in each segment's start and end position, but also the start position of the next segment, for three points total. This defines an oriented corner about the polyline. The SDF for two edges forming a single corner can be queried and merged pretty easily all at once by taking the mininum of each SDF.

The result are these perfectly antialiased and rounded polylines, all drawn with just a small number of tightly wrapped quads.

<div style="text-align: center;">
  <img src="/assets/aa_lines_translucent.gif" alt="aa_lines_translucent">
</div>

Bugs aside (apologies again for that) this style of polyline rendering handles all but self-intersecting cases of disparate corners. However, self-intersecting corners are gracefully handled due to SDF merging in the fragment shader. It's really an amazing strategy to get perfcetly rounded, antialiased lines going. You can even support annular/stroked lines (outlines with empty interiors) with this method without any additional code complexity. It gracefully handles thin lines less than a pixel in width (you can clamp to some visual distance, e.g. half a pixel width) all the way up to, and beyond, very fat lines spanning the screen.

Here's what they look like opaque:

<div style="text-align: center;">
  <img src="/assets/aa_lines.gif" alt="aa_lines">
</div>

# Other Shape Effects

Since the shape rendering is largely controlled by SDF's it's entirely possible to modulate rendering based on, for example, environmental factors. If a particular fragment is say, far away you can tone down transparency, blur, adjust color, or even offset positions based on a texture mask. Since rendering happens in the fragment shader the essence of the shape itself is available for tweaking, and plays very nicely generally speaking.