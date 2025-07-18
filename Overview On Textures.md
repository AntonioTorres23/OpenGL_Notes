
**Textures**

We learned that to add more detail to our objects we can use colors for each vertex to create some interesting images. However, to get a fair bit of realism we'd have to have many vertices so we could specify a lot of colors. This takes up a considerable amount of extra overhead, since each model needs a lot more vertices and for each vertex a color attribute as well. 

What artists and programmers generally prefer is to use a **texture**. A texture is a 2D image (even 1D and 3D textures exist) used to add detail to an object; think of a texture as a piece of paper with a nice brick image on it neatly folded over a 3D house so it looks like the house has a stone exterior. Because we can insert a lot of detail in a single image, we can give the illusion the object is extremely detailed without have to specify extra vertices. 

Next to images, textures can also be used to store a large collection of arbitrary data to send to the shaders, but we'll leave that for a different topic. 

Below you'll see a textured image of a [brick wall](https://learnopengl.com/img/textures/wall.jpg) mapped to a triangle. 

![[Pasted image 20250718161227.png]]

In order to map a texture to the triangle we need to tell each vertex of the triangle which part of the texture it corresponds to. Each vertex should thus have a **texture coordinate** associated with them that specifies what part of the textured image to sample from. Fragment interpolation then does the rest for the other fragments. 

Texture coordinates range from 0 to 1 in the x and y axis (remember that we use 2D texture images). Retrieving the texture color using texture coordinates is called **sampling**. Texture coordinates start at (0,0) for the lower left corner of a texture image to (1,1) for the upper right corner of a texture image. The following image shows how we map texture coordinates to the triangle. 

![[Pasted image 20250718161904.png]]

We specify 3 texture coordinate points for the triangle. We want the bottom-left 