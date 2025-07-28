
**Textures**

We learned that to add more detail to our objects we can use colors for each vertex to create some interesting images. However, to get a fair bit of realism we'd have to have many vertices so we could specify a lot of colors. This takes up a considerable amount of extra overhead, since each model needs a lot more vertices and for each vertex a color attribute as well. 

What artists and programmers generally prefer is to use a **texture**. A texture is a 2D image (even 1D and 3D textures exist) used to add detail to an object; think of a texture as a piece of paper with a nice brick image on it neatly folded over a 3D house so it looks like the house has a stone exterior. Because we can insert a lot of detail in a single image, we can give the illusion the object is extremely detailed without have to specify extra vertices. 

Next to images, textures can also be used to store a large collection of arbitrary data to send to the shaders, but we'll leave that for a different topic. 

Below you'll see a textured image of a [brick wall](https://learnopengl.com/img/textures/wall.jpg) mapped to a triangle. 

![[Pasted image 20250718161227.png]]

In order to map a texture to the triangle we need to tell each vertex of the triangle which part of the texture it corresponds to. Each vertex should thus have a **texture coordinate** associated with them that specifies what part of the textured image to sample from. Fragment interpolation then does the rest for the other fragments. 

Texture coordinates range from 0 to 1 in the x and y axis (remember that we use 2D texture images). Retrieving the texture color using texture coordinates is called **sampling**. Texture coordinates start at (0,0) for the lower left corner of a texture image to (1,1) for the upper right corner of a texture image. The following image shows how we map texture coordinates to the triangle. 

![[Pasted image 20250718161904.png]]

We specify 3 texture coordinate points for the triangle. We want the bottom-left side of the triangle correspond with the bottom-left side of the triangle texture so we use the (0,0) texture coordinate for the triangle's bottom-left vertex. The same applies to the bottom right side with a (1,0) texture coordinate. The top of the triangle should correspond with the top-center of the texture image so we take (0.5,1.0) as its texture coordinate. We only have to pass 3 texture coordinates to the vertex shader, which then passes those to the fragment shader that neatly interpolates all the texture coordinates for each fragment. 

The resulting texture coordinates would look like this:

```
float texCoords[] =
{
	0.0f, 0.0f, // lower-left corner
	1.0f, 0.0f, // lower-right corner
	0.5f, 1.0f  // top-center corner
};
```

Texture sampling has a loose interpretation and can be done in many ways. It is thus our job to tell OpenGL how it should *sample* its textures. 

**Texture Wrapping**

Texture coordinates usually range from (0,0) to (1,1) but what happens if we specify coordinates outside of this range? The default behavior of OpenGL is to repeat the textured images (we basically ignore the integer part of the floating point texture coordinate), but there are more options OpenGL offers.

- `GL_REPEAT`: The default behaviors for textures. Repeats the textured image. 
- `GL_MIRRORED_REPEAT`: Same as `GL_REPEAT` but mirrors the image with each repeat.
- `GL_CLAMP_TO_EDGE`: Clamps the coordinates between 0 and 1. The result is that higher coordinates become clamped to the edge, resulting in a stretched edge pattern. 
- `GL_CLAMP_TO_BORDER`: Coordinates outside the range are now given a user-specified border color. 

Each of the options have a different visual output when using texture coordinates outside the default range. Let's see what these look like on a sampled texture image. 

![[Pasted image 20250718164610.png]]

Each of the aforementioned options can be set per coordinate axis (s, t (and r if you're using 3D textures) equivalent to x, y, z) with the `glTexParameter*` function.

```
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT); //x texCoord
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT); //y texCoord
```

The first argument specifies the texture target; we're working with 2D textures so the texture target is `GL_TEXTURE_2D`. The second argument requires us to tell what option we want to set and for which texture axis; we want to configure it for both the S and T axis. The last argument requires us to pass in the texture wrapping mode we'd like and in this case OpenGL will set its texture wrapping option on the currently active texture with `GL_MIRRORED_REPEAT`.

If we chose the `GL_CLAMP_TO_BORDER` option we should also specify a border color. This is done using the `fv` equivalent of the `glTexParameter` function with `GL_TEXTURE_BORDER_COLOR` as its option where we pass in a float array of the border's color value:

```
float borderColor[] = { 1.0f, 1.0f, 0.0, 1.0f};
glTexParameteriv(GL_TEXTURE_2D, GL_CLAMP_TO_BORDER_COLOR, borderColor);
```

**Texture Filtering**

Texture coordinates do not depend on resolution but can be any floating point value, thus OpenGL has to figure out which texture pixel (also known as a **texel**) to map the texture coordinate to. This becomes especially important if you have a very large object and a low resolution texture. You probably guessed by now that OpenGL has options for this **texture filtering** as well. There are several options available but now we'll discuss the most important options: `GL_NEAREST` and `GL_LINEAR`. 

`GL_NEAREST` (also known as **nearest neighbor** or **point** filtering) is the default texture filtering method of OpenGL. When set to `GL_NEAREST`, OpenGL selects the texel that center is closest to the texture coordinate. Below you can see 4 pixels where the cross represents the exact texture coordinate. The upper-left texel has its center closest to the texture coordinate and is therefore chosen as the sampled color. 

![[Pasted image 20250725165801.png]]

`GL_LINEAR` (also known as (bi)linear filtering) takes an interpolated value from the texture coordinate's neighboring texels, approximating a color between the texels. The smaller the distance from the texture coordinate to a texel's center, the more that texel's color contributes to the sampled color. Below we can see that a mixed color of the neighboring pixels is returned.

![[Pasted image 20250728120106.png]]

But what is the visual effect of such a texture filtering method? Let's see how these methods work when using a texture with a low resolution on a large object (texture is therefore scaled upwards and individual texels are noticeable). 

![[Pasted image 20250728120520.png]]

`GL_NEAREST` results in blocked patterns where we can clearly see the pixels that form the texture while `GL_LINEAR` produces a smoother pattern where the individual pixels are less visible. `GL_LINEAR` produces a more realistic output, but some developers prefer a more 8-bit look and as a result pick the `GL_NEAREST` option. 

Texture filtering can be set for `magnifying` and `miniflying` operations (when scaling or downwards) so you could for example use nearest neighbor filtering when textures are scaled downwards and linear filtering for upscaled textures. We thus have to specify the filtering method for both options via `glTexParameter*`. The code should look similar to setting the wrapping method. 

`glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);`
`glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);`

**Mipmaps**

Imagine we had a large room with thousands of objects, each with an attached texture. There will be objects far away that have the same high resolution texture attached as the objects close to the viewer. Since the objects are far away and probably only produce a few fragments, OpenGL has difficulties retrieving the right color value for its fragment from the high resolution texture, since it has to pick a texture color for a fragment that spans a large part of the texture. This will produce visible artifacts on small objects, not to mention the waste of memory bandwidth using high resolution textures on small objects. 

To solve this issue OpenGL uses a concept called **mipmaps** that is basically a collection of texture images where each subsequent texture is twice as small compared to the previous one. The idea behind mipmaps should be easy to understand: after a certain distance threshold from the viewer, OpenGL will use a different mipmap texture that best suits the distance to the object. Because the object is far away, the smaller resolution will not be noticeable to the user. OpenGL is then able to sample correct texels, and there's less cache memory involved involved when sampling that part of the mipmaps. Let's take a closer look at what a mipmapped texture looks like. 

![[Pasted image 20250728152210.png]]

Creating a collection of mipmapped textures for each texture image is cumbersome to do manually, but luckily OpenGL is able to do all that work for us with a single call to `glGenerateMipmap` after we've created a texture. 

When switching between mipmaps levels during rendering OpenGL may show some artifacts like sharp edges visible between the two mipmap layers. Just like normal texture filtering, it is also possible to filter between mipmap levels using `NEAREST` and `LINEAR` filtering for switching between mipmap levels. To specify the filtering method between mipmap levels we can replace the original filtering methods with one of the four following options. 