
**Textures**

We learned that to add more detail to our objects we can use colors for each vertex to create some interesting images. However, to get a fair bit of realism we'd have to have many vertices so we could specify a lot of colors. This takes up a considerable amount of extra overhead, since each model needs a lot more vertices and for each vertex a color attribute as well. 

What artists and programmers generally prefer is to use a **texture**. A texture is a 2D image (even 1D and 3D textures exist) used to add detail to an object; think of a texture as a piece of paper with a nice brick image on it neatly folded over a 3D house so it looks like the house has a stone exterior. Because we can insert a lot of detail in a single image, we can give the illusion the object is extremely detailed without have to specify extra vertices. 

Next to images, textures can also be used to store a large collection of arbitrary data to send to the shaders, but we'll leave that for a different topic. 

Below you'll see a textured image of a [brick wall](https://learnopengl.com/img/textures/wall.jpg) mapped to a triangle. 

![[Pasted image 20250718161227.png]]

