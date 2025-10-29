
In the shadow mapping notes we learned to create dynamic shadows with shadow mapping. It works great, but it's mostly suited for directional (or spot) lights as the shadows are generated only in the direction of the light source. It is therefore also known as **directional shadow mapping** as the depth (or shadow) map is generated from only the direction the light is looking at. 

What this section of notes will focus on is the generation of dynamic shadows in all surrounding directions. The technique we're using is perfect for point lights as a real point light would cast shadows in all directions. This technique is known as point (light) shadows or more formerly as **omnidirectional shadow maps**. 

This section of notes builds upon the previous shadow mapping notes, so unless you're familiar with traditional shadow mapping it is advised to read the shadow mapping section first. 

This technique is mostly similar to directional shadow mapping: we generate a depth map from the light's perspective(s), sample the depth map based on the current fragment position, and compare each fragment with the stored depth value to see whether it is in shadow. The main difference between directional shadow mapping and omnidirectional shadow mapping is the depth map we use. 

The depth map we need requires a rendering scene from all surrounding directions of a point light and as such a normal 2D depth map will not work; what if we were to use a cubemap instead? Because a cubemap can store full environmental data with only 6 faces, it is possible to render the entire scene to each of the faces of a cubemap and sample these as the point light's surrounding depth values. 

![[Pasted image 20251029143137.png]]

The generated depth cubemap is then passed to the lighting fragment shader that samples the cubemap with a direction vector to obtain the closest depth (from the light's perspective) at that fragment. Most of the complicated stuff we've already discussed in the shadow mapping notes. What makes this technique a bit more difficult is the depth cubemap generation. 

**Generating the Depth Cubemap**

To create a cubemap of a light's surrounding depth values we have to render the scene 6 times: one for each face. One (quite obvious) way to do this, is render the scene 6 times with 6 different view matrices, each time attaching a different cubemap face to the framebuffer object. This would look something like this. 

```
for(unsigned int i = 0; i < 6 ; i++)
{
	Glenum face = GL_TEXTURE_CUBE_MAP_POSITIVE_X + i;
	glFramebuffer2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, face, depthCubemap, 0);
	BindViewMatrix(lightViewMatrices[i]);
	RenderScene();
}
```


This can be quite expensive though as a lot of render calls are necessary for this single depth map. In these notes we're going to use an alternative (more organized) approach using a little trick in the geometry shader that allows us to build the depth cubemap with just a single render pass. 

First we'll need to create a cubemap.

```
unsigned int depthCubemap;
glGenTextures(1, &depthCubemap);
```

And assign each of the single cubemap faces a 2D depth-valued texture image.

```
const unsigned int SHADOW_WIDTH = 1024, SHADOW_HEIGHT = 1024;
glBindTexture(GL_TEXTURE_CUBE_MAP, depthCubemap);
for (unsigned int i = 0; i < 6; i++)
{
	glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_DEPTH_COMPONENT,            SHADOW_WIDTH, SHADOW_HEIGHT, 0, GL_DEPTH_COMPONENT, GL_FLOAT, NULL);
}
```

And don't forget to set the texture parameters.

```
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE); 
```

Normally we'd attach a single face of a cubemap texture to the framebuffer object and render the scene 6 times, each time switching the depth buffer target of the framebuffer to a different cubemap face. Since we're going to use a geometry shader, that allows us to render all faces in a single pass, we can directly attach the cubemap as a framebuffer's depth attachment with `glFrameBufferTexture`.

```
glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
glFrameBufferTexture(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, depthCubemap, 0);
glDrawBuffer(GL_NONE);
glReadBuffer(GL_NONE);
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

Again, note the call to `glDrawFramebuffer` and `glReadFrameBuffer`: we only care about depth values when generating a depth cubemap so we have to explicitly tell OpenGL this framebuffer object does not render to a color buffer. 

With omnidirectional shadow maps we have two render passes: first, we generate the depth cubemap and second, we use the depth cubemap in the normal render pass to add shadows to the scene. This process looks like this. 

```
// 1. First render to depth cubemap
glViewport(0, 0, SHADOW_WIDTH, SHADOW_HEIGHT);
glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
	glClear(GL_DEPTH_BUFFER_BIT);
	ConfigureShaderAndMatrices();
	RenderScene();
glBindFramebuffer(GL_FRAMEBUFFER, 0);
// 2. Then, render the scene as normal with shadow mapping (using depth cubemap);
glViewport(0, 0, SCR_WIDTH, SCR_HEIGHT);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
ConfigureShaderAndMatrices();
glBindTexture(GL_TEXTURE_CUBE_MAP, depthCubemap);
RenderScene();
```

The process is exactly the same as with default shadow mapping, although this time we render to and use a cubemap depth texture compared to a 2D depth texture. 

**Light Space Transform**

With the framebuffer and cubemap set, we need some way to transform all the scene's geometry to the relevant light spaces in all 6 directions of the light. Just like the shadow mapping notes we're going to need a light space transformation matrix $T$, but this time one for each face. 

Each light space transf

