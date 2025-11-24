
The way we did lighting so far was called **forward rendering** or **forward shading**. A straightforward approach where we render an object and light it according to all the light sources in a scene. We do this for every object individually for the object in the scene. While quite easy to understand and implement it is also quite heavy on performance as each rendered object has to iterate over each light source for every rendered fragment, which is a lot! Forward rendering also tends to waste a lot of fragment shader runs in scenes with a high depth complexity (multiple objects cover the same screen pixel) as fragment shader outputs are overwritten. 

**Deferred Shading** or **deferred rendering** aims to overcomes these issues by drastically changing the way we render objects. This gives us several new options to significantly optimize scenes with large numbers of lights. The following image is a scene with 1847 point lights rendered with deferred shading (image courtesy of Hannes Nevalainen); something that wouldn't be possible with forward rendering. 

![[Pasted image 20251124112444.png]]

Deferred shading is based on the image that we *defer* or *postpone* most of the heavy rendering (like lighting) to a later stage. Deferred shading consists of two passes: in the first pass, called the **geometry pass**, we render the scene once and retrieve all kinds of geometrical information from the objects that we store in a collection of textures called the **G-buffer**; think of position vectors, color vectors, normal vectors, and/or specular values. The geometric information of a scene stored in the **G-buffer** is then later used for (more complex) lighting calculations. Below is the content of a G-buffer of a single frame. 

![[Pasted image 20251124113627.png]]

We use the textures from the G-buffer in a second pass called the **lighting pass** where we render a screen-filled quad and calculate the scene's lighting for each fragment using geometrical information stored in the G-buffer; pixel by pixel we iterate over the G-buffer. Instead of taking each object all the way from the vertex shader to the fragment shader, we decouple its advanced fragment processes to a later state. The lighting calculations are exactly the same, but this time we take all required input variables from the corresponding G-buffer textures, instead of the vertex shader (plus some uniform variables). 

The image below nicely illustrates the process of deferred shading. 

![[Pasted image 20251124115502.png]]

A major advantage of this approach is that whatever fragment ends up in the G-buffer is the actual fragment information that ends up as a screen pixel. The depth test already concluded this fragment to be the last and top-most fragment. This ensures that for each pixel we process in the lighting pass, we only calculate lighting once. Furthermore, deferred rendering opens up the possibility for further optimizations that allow us to render a much larger amount of light sources compared to forward rendering. 

In also comes with some disadvantages though as the G-buffer requires us to store a relatively large amount of scene data in its texture color buffers. This eats memory, especially since scene data like position vectors require a high precision. Another disadvantage is that it doesn't support blending (as we only have information of the top-most fragment) and MSAA no longer works. There are several workarounds for this that we'll get to at the end of the notes. 

Filling the G-buffer (in the geometry pass) isn't too expensive as we directly store object information like position, color, or normals into a framebuffer with a small or zero amount of processing. By using **multiple render targets (MRT)** we can even do all of this in a single render pass. 

**The G-buffer**

The **G-buffer** is the collective term of all textures used to store lighting-relevant data for the final lighting pass. Let's take this moment to briefly review all the data we need to light a fragment with forward rendering. 

