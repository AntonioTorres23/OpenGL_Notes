
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

