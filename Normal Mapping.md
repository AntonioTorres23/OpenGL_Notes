
All of our scenes are filled with meshes, each consisting of hundreds of maybe thousands of triangles. We boosted the realism by wrapping 2D textures on these flat triangles, hiding the fact that the polygons are just tiny flat triangles. Textures help, but when you take a good close look at the meshes it is still quite easy to see the underlying flat surfaces. Most real-life surfaces aren't flat however and exhibit a lot of (bumpy) details. 

For instance, take a brick surface. A brick surface is quite a rough surface and obviously not completely flat: it contains sunken cement stripes and a lot of detailed little holes and cracks. If we were to view such a brick surface in a lit scene the immersion gets easily broken. Below we can see a brick texture applied to a flat surface lit by a point light. 

![[Pasted image 20251105142709.png]]

The lighting doesn't take any of the small cracks and holes into account and completely ignores the deep stripes between the bricks; 