
All of our scenes are filled with meshes, each consisting of hundreds of maybe thousands of triangles. We boosted the realism by wrapping 2D textures on these flat triangles, hiding the fact that the polygons are just tiny flat triangles. Textures help, but when you take a good close look at the meshes it is still quite easy to see the underlying flat surfaces. Most real-life surfaces aren't flat however and exhibit a lot of (bumpy) details. 

For instance, take a brick surface. A brick surface is quite a rough surface and obviously not completely flat: it contains sunken cement stripes and a lot of detailed little holes and cracks. If we were to view such a brick surface in a lit scene the immersion gets easily broken. Below we can see a brick texture applied to a flat surface lit by a point light. 

![[Pasted image 20251105142709.png]]

The lighting doesn't take any of the small cracks and holes into account and completely ignores the deep stripes between the bricks; the surface looks perfectly flat. We can partly fix the flat look by using a specular to pretend some surfaces are less lit due to depth or other details, but that's more of a hack than a real solution. What we need is some way to inform the lighting system about all the little depth-like details of the surface. 

If we think about this from a light's perspective: how come the surface is lit as a completely flat surface? The answer is the surface's normal vector. From the lighting technique's point of view, the only way it determines the shape of an object is by its perpendicular normal vector. The brick surface only has a single normal vector, and as a result the surface is uniformly lit based on this normal vector's direction. What if we, instead of a per-surface normal that is the same for each fragment, use a per-fragment normal that is different for each fragment? This way we can slightly deviate the normal vector based on a surface's little details; this gives the illusion the surface is a lot more complex.

![[Pasted image 20251105162447.png]]
By using per-fragment normals we can trick the lighting into believing a surface consists of tiny little planes (perpendicular to the normal vectors) giving the surface an enormous boost in detail. This technique to use per-fragment normals compared to per-surface normals is called **normal** **mapping** or **bump mapping**. Applied to the brick plane it looks a bit like this.  

![[Pasted image 20251105164243.png]]

As you can see, it gives an enormous boost in detail and for a relatively low cost. Since we only change the normal vectors per fragment there is no need to change the lighting equation. We now pass a per-fragment normal, instead of an interpolated surface normal, to the lighting algorithm. The lighting then does the rest. 

**Normal Mapping**

To get normal mapping to work we're going to need a per-fragment normal. Similar to what we did diffuse and specular maps we can use a 2D texture to store per-fragment normal data. This way we can sample a 2D texture to get a normal vector for that specific fragment. 

While normal vectors are geometric entities and textures are generally only used for color information, storing normal vectors in a texture may not be immediately obvious. If you think about color vectors in a texture they are represented as a 3D vector with an r, g, and b component. We can similarly store a normal vector's x, y, and z component in the respective color component. Normal vectors range between -1 and 1 so they're first mapped to $[0,1]$.
`// I think we use convert the normal from [-1, 1] to [0, 1] is becuase`
`// a `
`vec3 rgb_normal = normal * 0.5 + 0.5; // transforms from [-1,1] to [0,1]`

With normal vectors transformed to an RGB color component like this, we can store a per-fragment normal derived from the shape of a surface onto a 2D texture. An example **normal map** of the brock surface at the start of this chapter is shown below. 

![[Pasted image 20251107125446.png]]

This (and almost all normal maps you find online) will have a blueish tint. This is because the normals are all closely pointing outwards towards the positive z axis $(0, 0, 1)$: a blueish color. The deviations in color represent normal vectors that are slightly offset from the general positive z direction, giving a sense of depth to the texture. For example, you can see that at the top of each brick the color tends to be more greenish, which makes sense as the top side of a brick would have normals pointing more in the positive y direction $(0, 1, 0)$ which happens to be the color green.

With a simple plane, looking at the positive z-axis, we can take [this](https://learnopengl.com/img/textures/brickwall.jpg) diffuse texture and [this](https://learnopengl.com/img/textures/brickwall_normal.jpg) normal map to render the image from the previous section. Note that the linked normal map is different from the one shown above. The reason for this is that OpenGL reads texture coordinates with the y (or v) coordinate reversed from how textures are generally created. The linked normal map thus has its y (or green) component inversed 

