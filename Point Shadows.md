
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

Each light space transformation matrix contains both a projection and view matrix. For the projection matrix we're going to use a perspective projection matrix (contains fov); the light source represents a point in space so perspective projection makes most sense. Each light space transformation matrix uses the same projection matrix. 

```
float aspect = (flaot)SHADOW_WIDTH/(float)SHADOW_HEIGHT; // gets aspect ratio
float near = 1.0f; // near-plane of perspective matrix
float far = 25.0f; // far-plane of perspective matrix
// don't forget to set the Field of View to radians
glm::mat4 shadowProj = glm::perspective(glm::radians(90.0f), aspect, near, far);
```

Important to note here is the field of view parameter of `glm::perspective` that we set to 90 degrees. By setting this to 90 degrees we make sure the viewing field is exactly large enough to fill  a single face of the cubemap such that all faces align correctly to each other at the edges. 

As the projection matrix does not change per direction we can re-use it for each of the 6 transformation matrices. We do need a different view matrix per direction. With `glm::lookAt`
we create 6 view-directions, each looking at one face direction of the cubemap in order: right, left, top, bottom, front (near), back (far). 

```
std::vector<glm::mat4> shadowTransforms;
shadowTransforms.push_back(shadowProj * glm::lookAt(lightPos, lightPos + glm::vec3(1.0, 0.0, 0.0), glm::vec3(0.0, 1.0, 0.0));
shadowTransforms.push_back(shadowProj * glm::lookAt(lightPos, lightPos + glm::vec3(-1.0, 0.0, 0.0), glm::vec3(0.0, 1.0, 0.0));
shadowTransforms.push_back(shadowProj * glm::lookAt(lightPos, lightPos + glm::vec3(0.0, 1.0, 0.0), glm::vec3(0.0, 1.0, 0.0));
shadowTransforms.push_back(shadowProj * glm::lookAt(lightPos, lightPos + glm::vec3(0.0, -1.0, 0.0), glm::vec3(0.0, 1.0, 0.0));
shadowTransforms.push_back(shadowProj * glm::lookAt(lightPos, lightPos + glm::vec3(0.0, 0.0, 1.0), glm::vec3(0.0, 1.0, 0.0));
shadowTransforms.push_back(shadowProj * glm::lookAt(lightPos, lightPos + glm::vec3(0.0, 0.0, -1.0), glm::vec3(0.0, 1.0, 0.0));
```

Here we create 6 view matrices and multiply them with the projection matrix to get a total of 6 different light space transformation matrices. The `target` parameter of `glm::lookAt` each looks into the direction of a single cubemap face. 

These transformation matrices are sent to the shaders that render the depth into the cubemap. 

**Depth Shaders**

To render depth values to a depth cubemap we're going to need a total of 3 shaders: a vertex, a geometer, and a fragment shader. 

The geometry shader will be the shader responsible for transforming all world-space vertices to the 6 different light spaces. Therefore, the vertex shader simply simply transforms vertices to world-space and directs them to the geometry shader. 

```
#version 330 core
layout (location = 0) in vec3 aPos

uniform mat4 model;

void main()
{
	gl_Position = model * vec4(aPos, 1.0);
}
```

The geometry shader will take as input 3 triangle vertices and a uniform array of light space transformation matrices. The geometry shader is responsible for transforming the vertices to the light spaces; this is also where it gets interesting. 

The geometry shader has a built-in variable called `gl_Layer` that specifies which cubemap face to emit a primitive to. When left alone, the geometry shader just sends its primitives further down the pipeline as usual, but when we update this variable we can control which cubemap face we render to for each primitive. This of course only works when we have a cubemap texture attached to the active framebuffer. 

```
#version 330 core
layout (triangles) in;
layout (trianle_strip max_vertices=18) out;

uniform mat4 shadowMatrices[6];

out vec4 FragPos; // FragPos from GS (output per emitvertex)

void main()
{
	for(int face = 0; face < 6; ++face)
	{
		gl_Layer = face; // built-in variable that specifices to which face we                              //                                            render
		for(int i = 0; i < 3, ++i)
		{
			FragPos = gl_in[i].gl_Position;
			gl_Position = shadowMatrices[face] * FragPos;
			EmitVertex();
		}
		EndPrimitive
	}
}
```

This geometry shader is relatively straightforward. We take as input a triangle, and output a total of 6 triangles (6 * 3 equals 18 vertices). In the main function we iterate over 6 cubemap faces where we specify each face as the output face by storing the face integer into `gl_Layer`. We then generate the output triangles by transforming each world-space input vertex to the relevant light space by multiplying `FragPos` with the face's light-space transformation matrix. Note that we also send the resulting `FragPos` variable to the fragment shader that we'll need to calculate a depth value. 

In the last notes we use an empty fragment shader and let OpenGL figure out the depth values of the depth map. This time we're going to calculate our own (linear) depth as the linear distance between each closest fragment position and the light source's position. Calculating our own depth values makes the later shadow calculations a bit more intuitive. 

```
#version 330 core
in vec4 FragPos;

uniform vec3 lightPos;
uniform float far_plane;

void main()
{
	// get distance between fragment and light source
	float lightDistance = length(FragPos.xyz - lightPos);
	
	// map to [0-1] range by dividing by far-plane
	lightDistance = lightDistance / far_plane;
	
	// write this as modified depth
	gl_FragDepth = lightDistance;
}
```

The fragment shader takes as input the `FragPos` from the geometry shader, the light's position vector, and the frustum's far plane value. Here we take the distance between the fragment and the light source, map it to the $[0,1]$  range and write it as the fragment's depth value. 

Rendering the scene with these shaders and the cubemap-attached framebuffer object active should give you a completely filled depth cubemap for the second pass's shadow calculations.

I believe the fragment shader is only for debugging purposes as to see the actual depth values from the cubemap.

**Omnidirectional Shadow Maps**

With everything set up it is time to render the actual omnidirectional shadows. The procedure is similar to the directional shadow mapping notes, although this time we bind a cubemap texture instead of a 2D texture and also pass the light projection's far plane variable to the shaders. 

```
glViewPort(0, 0, SCR_WIDTH, SCR_HEIGHT);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
shader.use()
// ... send uniforms to shader (including light's far_plane value)
glActivateTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_CUBE_MAP, depthCubemap);
// ... bind other textures
RenderScene();
```

Here the `renderScene` function renders a few cubes in a large room scatted around a light source at the center of the scene. 

The vertex and fragment shader are mostly similar to the original shadow mapping shaders: the difference being that the fragment shader no longer requires a fragment position in light space as we can now sample the depth values with a direction vector. 

Because of this, the vertex shader doesn't needs to transform its position vectors to light space so we can remove the `FragPosLightSpace` variable. 

```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoords;

out vec2 TexCoords;

out VS_OUT {
	vec3 FragPos;
	vec3 Normal;
	vec2 TexCoords;
	
} vs_out;

uniform mat4 projection;
uniform mat4 view; 
uniform mat4 model;

void main()
{
	vs_out.FragPos = vec3(model * vec4(aPos, 1.0));
	vs_out.Normal = transpose(inverse(mat3(model))) * aNormal;
	vs_out.TexCoords = aTexCoords;
	
	gl_Position = projection * view * model * (aPos, 1.0);
}
```

The fragment shader's Blinn-Phong lighting code is exactly the same as we had before with a shadow multiplication at the end. 

```
#version 330 core
out vec4 FragColor;

in VS_OUT {
	vec3 FragPos;
	vec3 Normal;
	vec2 TexCoords;
} fs_in

uniform sampler2D diffuseTexture;
uniform samplerCube depthMap;

uniform vec3 lightPos;
uniform vec3 viewPos;

uniform float far_plane;

float ShadowCalculations(vec3 fragPos)
{
	[...]
}

void main()
{
	vec3 color = texture(diffuseTexture, fs_in.TexCoords);
	vec3 normal = normalize(fs_in.Normal);
	vec3 lightColor = vec3(0.3);
	// ambient
	vec3 ambient = color * 0.3;
	// diffuse
	vec3 lightDir = normalize(lightPos - fs_in.FragPos);
	float diff = max(dot(lightDir, normal), 0.0);
	vec3 diffuse = diff * lightColor; 
	// specular 
	vec3 viewDir = normalize(viewPos - fs_in.FragPos);
	float spec = 0.0;
	vec3 halfwayDir = normalize(lightDir + viewDir);
	spec = pow(max(dot(normal, halfwayDir), 0.0), 64.0);
	vec3 specular = spec * lightColor;
	// calculate shadow
	float shadow = ShadowCalculations(fs_in.FragPos);	

	vec3 lighting = (ambient + (1.0 - shadow) * (diffuse + specular)) * color;
	
	FragColor = vec4(lighting, 1.0);
}

```

There are a few subtle differences: the lighting code is the same, but we now have a `samplerCube` uniform and the `ShadowCalculation` takes the current fragment's positions as its argument instead of the fragment position in light space. We also include the light's frustum `far_plane` that we'll later need. 

The biggest difference is in the content of the `ShadowCalculation` function that now samples depth values from a cubemap instead of a 2D texture. Let's discuss its content step by step. 

The first thing we have to do is retrieve the depth of the cubemap. You may remember from the cubemap section of these notes that we stored the depth as the linear distance between the fragment and the light position; we're taking a similar approach here. 

```
float ShadowCalculation(vec3 fragPos)
{
	vec3 fragToLight = fragPos - lightPos;
	float closestDepth = texture(depthMap, fragToLight).r;
}
```

Here we take the difference vector between the fragment's position and the light's position and use that vector as a direction vector to sample the cubemap. The direction vector doesn't need to be a unit vector to sample from a cubemap so there's no need to normalize it. The resulting `closestDepth` value is the normalized depth value between the light source and its closest visible fragment. 

The `closestDepth` value is currently in the range $[0,1]$ so we first transform it back to $[0, far plane]$ by multiplying it with the `far_plane`.

`closestDepth *= far_plane;`

Next we retrieve the depth value between the current fragment and the light source, which we can easily obtain by taking the length of `fragToLight` due to how we calculated depth values in the cubemap.

`float currentDepth = length(fragToLight);`

This returns a depth value in the same (or larger) range as `closestDepth`. 

Now we can compare both depth values to see which is closer than the other and determine whether the current fragment is in shadow. We also include a shadow bias so we don't get shadow acne as discussed in the shadow mapping notes. 

`float bias = 0.05;`
`float shadow = currentDepth - bias > closestDepth ? 1.0 : 0.0;`

The complete `ShadowCalculation` then becomes. 

```
float ShadowCalculations(vec3 fragPos)
{
	// get vector between fragment position and light position
	vec3 fragToLight = fragPos - lightPos;
	// use the light to fragment vector to sample from the depth map
	float closestDepth = texture(depthMap, fragToLight).r; 
	// it is currently in linear range between [0,1]. Re-transform back to original
	// value
	closestDepth *= far_plane;
	// now get current linear depth as the length between the fragment and light 
	// position
	float currentDepth = length(fragToLight);
	// test for shadows
	float bias = 0.05;
	float shadow = currentDepth - bias > closestDepth ? 1.0 : 0.0;
	
	return shadow;	
}
```


With these shaders we already get pretty good shadows and this time in all surrounding directions from a point light. With a point light positioned at the center of a simple scene it'll look a bit like this. 

![[Pasted image 20251029170234.png]]

You can find the source code of this demo [here](https://learnopengl.com/code_viewer_gh.php?code=src/5.advanced_lighting/3.2.1.point_shadows/point_shadows.cpp).

**Visualizing Cubemap Depth Buffer**

If you're somewhat like me you probably didn't get this right on the first try so it makes sense to do some debugging, with one of the obvious checks being validating whether the depth map was built correctly. A simple trick to visualize the depth buffer is to take the `closestDepth` variable in the `ShadowCalculation` function and display that variable as. 

```
FragColor = vec4(vec3(closestDepth / far_plane), 1.0);
```

The result is a grayed out scene where each color represents the linear depth values of the scene.

![[Pasted image 20251030094755.png]]


You can also see the to-be shadowed regions on the outside wall. If it looks somewhat similar, you know the depth cubemap was properly generated. 

**PCF**

Since omnidirectional shadow maps are based on the same principles as traditional shadow mapping it also has the same resolution dependent artifacts. If you zoom in close enough you can again see the jagged edges. **Percentage-closer Filtering** or PCF allows us to smooth out these jagged edges by filtering multiple samples around the fragment position and average the results. 

If we take the same simple PCF filter from the shadow mapping notes and add a third dimension we get. 

```
float shadow = 0.0;
float bias = 0.05;
float samples = 4.0;
float offset = 0.1;

for(float x = -offset; x < offset; x += offset / (samples * 0.5))
{
	for(float y = -offset; y < offset; y += offset / (samples * 0.5))
	{
		for(float z = -offset; z < offset; z += offset / (samples * 0.5))
		{
			float closestDepth = texture(depthMap, fragToLight + vec3(x, y, z)).r;
			closestDepth *= far_plane; // undo mapping [0,1]
			if(currentDepth - bias > closestDepth)
				s
		}
	}
}
```
















































































