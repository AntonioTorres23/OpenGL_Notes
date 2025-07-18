
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

The first argument specifies the texture target; we're working with 2D textures so the texture target is `GL_TEXTURE_2D`. The second argument requires us to tell what option we want to set and for which texture axis; we want to configure it for both the S and T axis. The last argument requires us to pass in the texture wrapping mode we'd like and in this case OpenGL will set its texture wrapping option on the currently active texture with G