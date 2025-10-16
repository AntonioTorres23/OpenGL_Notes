
Somewhere in your adventurous rendering journey you probably came across some jagged saw-like patterns along the edges of your models. The reason these **jagged edges** appear appear is due to how the rasterizer transforms the vertex data into actual fragments behind the scene. An example of what these jagged edges look like can already be seen when drawing a simple cube. 

![[Pasted image 20251016104626.png]]

While not immediately visible, if you take a closer look at the edges of the cube you'll see a jagged pattern. If we zoom in you'd see the following.

![[Pasted image 20251016104950.png]]

This is clearly not something we want in a final version of an application. This effect, of clearly seeing the pixel formations an edge is composed of, is called **aliasing**. There are quite a few techniques out there called **anti-aliasing** techniques that fight this aliasing behavior by producing *smoother* edges. 

At first we had a technique called **super sample anti-aliasing** (SSAA) that temporarily uses a much higher resolution render buffer to render the scene in (super sampling). 