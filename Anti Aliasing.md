
Somewhere in your adventurous rendering journey you probably came across some jagged saw-like patterns along the edges of your models. The reason these **jagged edges** appear appear is due to how the rasterizer transforms the vertex data into actual fragments behind the scene. An example of what these jagged edges look like can already be seen when drawing a simple cube. 

![[Pasted image 20251016104626.png]]

While not immediately visible, if you take a closer look at the edges of the cube you'll see a jagged pattern. If we zoom in you'd see the following.

![[Pasted image 20251016104950.png]]

This is clearly not something we want in a final version of an application. This effect, of clearly seeing the pixel formations an edge is composed of, is called **aliasing**. There are quite a few techniques out there called **anti-aliasing** techniques that fight this aliasing behavior by producing *smoother* edges. 

At first we had a technique called **super sample anti-aliasing** (SSAA) that temporarily uses a much higher resolution render buffer to render the scene in (super sampling). Then when the full scene is rendered, the resolution is downsampled back to the normal resolution. This *extra* resolution was used to prevent these jagged edges. While it did provide us with a solution to the aliasing problem, it came with a major performance drawback since we have to draw **a lot** more fragments than usual. This technique therefore only had a short glory moment. 

This technique did give birth to a more modern technique called **multisample anti-aliasing** or MSAA that borrows from the concepts behind SSAA while implementing a much more efficient approach. In this section of notes we'll be extensively discussing this MSAA technique that is built-in OpenGL. 

**Multisampling**

To understand what multisampling is and how it works into solving the aliasing problem we first need to delve a bit further into the inner workings of OpenGL's rasterizer. 

The rasterizer is the combination of all algorithms and processes that sit between your final processed vertices and the fragment shader. The rasterizer takes all vertices belonging to a single primitive and transforms this to a set of fragments. Vertex coordinates can theoretically have any coordinate, but fragments can't since they are bound by the resolution of your screen. There will almost never be a one-on-one mapping between vertex coordinates and fragments, so the rasterizer has to determine in some way what fragment/screen-coordinate each specific vertex will end up at. 

![[Pasted image 20251016122745.png]]

Here we see a grid of screen pixels where the center of each pixel contains a **sample point** that is used to determine if a pixel is covered by the triangle. The red sample points are covered by the triangle and a fragment will be generated for that covered pixel. Even though some parts of the triangle edges still enter certain screen pixels, the pixel's sample point is not covered by the inside of the triangle so this pixel won't be influenced by any fragment shader. 

You can probably already figure out the origin of aliasing now. The complete rendered version of the triangle would look like this on your screen. 

![[Pasted image 20251016150834.png]]

Due to the limited amount of screen pixels, some pixels will be rendered along an edge and some won't. The result is that we're rendering primitives with non-smooth edges giving rise to the jagged edges we've seen before. 

What multisampling does, is not use a single sampling point for determining coverage of the triangle, but multiple sample points (guess where it got its name from). Instead of a single sample point at the center of each pixel we're going to place 4 **subsamples** in a general pattern and use those to determine pixel coverage. 

![[Pasted image 20251016151220.png]]

The left side of the image shows how we would normally determine the coverage of a triangle. This specific pixel won't run a fragment shader (and thus remains blank) since its sample point wasn't covered by the triangle. The right side of the image shows a multisampled version where each pixel contains 4 sample points. Here we can see that only 2 of the sample points cover the triangle. 

The amount of sample points can be any number we'd like with more samples giving us better coverage precision. 

This is where multisampling becomes interesting. We determined that 2 subsamples were covered by the triangle so the next step is to determine a color for this specific pixel. Our initial guess would be that we run the fragment shader for each covered subsample and later average the colors of each subsample per pixel. In this case we'd run the fragment shader twice on the interpolated vertex data at each subsample and store the resulting color in those sample points. This is (fortunately) **not** how it works, because this would mean we need to run a lot more fragment shaders than without multisampling, drastically reducing performance. 

How MSAA really works is that the fragment shader is only run **once** per pixel (for each primi)