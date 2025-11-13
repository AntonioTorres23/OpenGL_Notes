
Parallax mapping is a technique similar to normal mapping, but based on different principles. Just like normal mapping it is a technique that significantly boosts a textured surface's detail and gives it a sense of depth. 

While also an illusion, parallax mapping is a lot better in conveying a sense of depth and together with normal mapping gives incredibly realistic results. While parallax mapping isn't necessarily a technique directly related to (advanced) lighting, I'll still discuss it here as the technique is a logical follow-up to normal mapping. Note that getting an understanding of normal mapping, specifically tangent space, is strongly advised before learning parallax mapping.  

Parallax mapping is closely related to the family of **displacement mapping** techniques that *displace* or *offset* vertices based on geometrical information stored inside a texture. One way to do this, is to take a plane with roughly 1000 vertices and displace each of these vertices based on a value in a texture that tells us the height of the plane at that specific area. Such a texture that contains height values per texels is called a **height map**. An example height map derived from the geometric properties of a simple brick surface looks a bit like this. 

![[Pasted image 20251113121351.png]]

When spanned over a plane, each vertex is displaced based on the sampled height value in the height map, transforming a plane to a rough bumpy surface based on a material's geometric properties. For instance, taking a flat plane displaced with the above heightmap results in the following image.

![[Pasted image 20251113143830.png]]

A problem with displacing vertices this way is that a plane needs to contain a huge amount of triangles to get a realistic displacement, otherwise the displacement looks too blocky. As each flat surface may then require over 10000 vertices. What if we could somehow achieve similar realism without the need of extra vertices? In fact, what if I were to tell you that the previously shown displaced surface is actually rendered with only 2 triangles. This brick surface show is rendered with **parallax mapping**, a displacement mapping technique that doesn't require extra vertex data to convey depth, but (similar to normal mapping) uses a clever technique to trick the user. 

The idea behind parallax mapping is to alter the texture coordinates in such a way that it looks like a fragment's surface is higher or lower than it actually is, all based on the view direction and a heightmap. To understand how it works, take a look at the following image of our brick surface. 

![[Pasted image 20251113144909.png]]

Here the rough red line represents values in the heightmap as the geometric surface representation of the brick surface and the vector $\color{orange}{\bar{V}}$ represents the surface to view direction (`viewDir`). If the plane would have actual displacement, the viewer would see the surface at point $\color{lightblue}{B}$. However, as our plane has no actual displacement the view direction is calculated from point $\color{green}{A}$ as we'd expect. Parallax mapping aims to offset the texture coordinates at fragment position $\color{green}{A}$ in such a way that we get texture coordinates at point $\color{lightblue}{B}$ for all subsequent texture samples, making it look like the viewer is actually looking at point $\color{lightblue}{B}$. We then use the texture coordinates at point $\color{lightblue}{B}$ for all subsequent texture samples, making it look like the viewer is actually looking at point $\color{lightblue}{B}$. 

The trick is to figure out how to get the texture coordinates at point $\color{lightblue}{B}$ from point $\color{green}{A}$. Parallax mapping tries to solve this by scaling the fragment-to-view direction vector $\color{orange}{\bar{V}}$ by the height at fragment $\color{green}{A}$. So we're scaling the length of $\color{orange}{\bar{V}}$ to be equal to a sampled value from the heightmap $\color{green}{H(A)}$ at fragment position $\color{green}{A}$. The image below shows this scaled vector $\color{brown}{\bar{P}}$. 


![[Pasted image 20251113150758.png]]


We then take this vector $\color{brown}{\bar{P}}$ and take its vector coordinates that align with the plane as the texture coordinate offset. This works because vector $\color{brown}{\bar{P}}$ is calculated using a height value from the heightmap. So the higher a fragment's height, the more it effectively gets displaced. 

This little trick gives good results most of the time, but it is still a really crude approximation to get to point $\color{lightblue}{B}$. When heights change rapidly over a surface the results tend to look unrealistic as the vector $\color{brown}{\bar{P}}$ will not end up close to $\color{lightblue}{B}$ as you can see below. 

![[Pasted image 20251113151518.png]]

Another issue with parallax mapping is that it's difficult to figure out which coordinates to retrieve from $\color{brown}{\bar{P}}$ when the surface is arbitrarily rotated in some way. We'd rather do this in a different coordinate space where the $x$ and $y$ component of vector $\color{brown}{\bar{P}}$ always align with the texture's surface. If you've followed along in the normal mapping notes you probably guessed how we can accomplish this. And yes, we would like to do parallax mapping in tangent space. 

By transforming the fragment-to-view vector $\color{orange}{\bar{V}}$ to tangent space, the transformed $\color{brown}{\bar{P}}$ vector will have its $x$ and $y$ component aligned to the surface's tangent and bitangent vectors. As the tangent and bitangent vectors are pointing in the same direction as the surface's texture coordinates we can take the $x$ and $y$ components of $\color{brown}{\bar{P}}$ as the texture coordinate offset, regardless of the surface's orientation. 

But enough about the theory, let's get our feet wet and start implementing parallax mapping.

**Parallax Mapping**

For parallax mapping we're going to use a simple 2D plane for which we calculated its tangent and bitangent vectors before sending it to the GPU; similar to what we did in the normal mapping notes. Onto the plane we're going to attach a [diffuse texture](https://learnopengl.com/img/textures/bricks2.jpg), a [normal map](https://learnopengl.com/img/textures/bricks2_normal.jpg), and a [displacement map](https://learnopengl.com/img/textures/bricks2_disp.jpg) that you can download from their URLs. For this example, we're going to use parallax mapping in conjunction with normal mapping. Because parallax mapping gives the illusion of displacing a surface, the illusion breaks when the lighting doesn't match. As normal maps are often generated from heightmaps, using a normal map together with a heightmap makes sure the lighting is in place with the displacement. 

You may have already noted that the displacement map lined above is inverse of the heightmap shown at the start of these notes. With parallax mapping it makes more sense to use the inverse of the heightmap as it's easier to fake depth than height on flat surfaces. This slightly changes how we perceive parallax mapping as show below. 

![[Pasted image 20251113154213.png]]
We again have points $\color{green}{A}$ and $\color{lightblue}{B}$, but this time we obtain vector $\color{brown}{\bar{P}}$ by **subtracting** vector $\color{orange}{\bar{V}}$ from texture coordinates at point $\color{green}{A}$. We can obtain depth values instead of height values by subtracting the sampled heightmap values from 1.0 in the shaders, or by simply inversing its texture values in image-editing software as we did with the depth map linked above. 

Parallax mapping is implemented in the fragment shader as the displaced effect is different all over a triangle's surface. In the fragment shader we're then going to need to calculate the fragment-to-view direction vector $\color{orange}{\bar{V}}$ so we need the view position and fragment position in tangent space. In the normal mapping notes we already had a vertex shader that sends these vectors in tangent space so we can take an exact copy of that note's vertex shader. 

```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoords;
layout (location = 3) in vec3 aTangent;
layout (location = 4) in vec3 aBitangent;

out VS_OUT {
	vec3 FragPos;
	vec2 TexCoords;
	vec3 TangentLightPos;
	vec3 TangentViewPos;
	vec3 TangentFragPos;
	
} vs_out;

uniform mat4 projection;
uniform mat4 view;
uniform mat4 model;

uniform vec3 lightPos;
uniform vec3 viewPos;

void main()
{
	gl_Position = projection * view * model * vec4(aPos, 1.0);
	
	vs_out.FragPos = vec3(model * vec4(aPos, 1.0));
	
	vs_out.TexCoords = aTexCoords; 
	
	vec3 T = normalize(mat3(model) * aTangent);
	vec3 B = normalize(mat3(model) * aBitangent);
	vec3 N = normalize(mat3(model) * aNormal);
	mat3 TBN = transpose(mat3(T,B,N));
	
	vs_out.TangentLightPos = TBN * lightPos;
	vs_out.TangentViewPos = TBN * viewPos;
	vs_out.TangentFragPos = TBN * vs_out.FragPos;
}
```

Within the fragment shader we then implement the parallax mapping logic. The fragment shader looks a bit like this. 

```
#version 330 core
out vec4 FragColor;

in VS_OUT {
	vec3 FragPos;
	vec2 TexCoords;
	vec3 TangentLightPos;
	vec3 TangentViewPos;
	vec3 TangentFragPos;
} fs_in;

uniform sampler2D diffuseMap;
uniform sampler2D normalMap;
uniform sampler2D depthMap;

uniform float height_scale;

vec2 ParallaxMapping(vec2 texCoords, vec3 viewDir);

void main()
{
	// offset texture coordinates with Parallax Mapping
	vec3 viewDir = normalize(fs_in.TangentViewPos - fs_in.TangentFragPos);
	vec2 texCoords = ParallaxMapping(fs_in.TexCoords, viewDir);
	
	// then sample textures with new texture coords
	vec3 diffuse = texture(diffuseMap, texCoords);
	vec3 normal = texture(normalMap, texCoords);
	// proceed with lighting code
	[...]
}

```

We defined a function called `ParallaxMapping` that takes as input the fragment's texture coordinates and the fragment-to-view direction $\color{orange}{\bar{V}}$ in tangent space. The function returns the displaced texture coordinates. We then use these displaced texture coordinates for sampling the diffuse and normal map. As a result, the fragment's diffuse and normal vector correctly corresponds to the surface's displaced geometry. 

Let's take a look inside the `ParallaxMapping` function. 

```
vec2 ParallaxMapping(vec2 texCoords, vec3 viewDir)
{
	float height = texture(depthMap, texCoords).r;
	vec2 p = viewDir.xy / viewDir.z * (height * height_scale)
	return texCoords - p; 
}
```


This relatively simple function is a direct translation of what we've discussed so far. We take the original texture coordinates `texCoords` and use these to sample the height (or depth) from the `depthMap` at the current fragment $\color{green}{A}$ as $\color{green}{H(A)}$. We then calculate $\color{brown}{\bar{P}}$ as the $x$ and $y$ component of the tangent-space `viewDir` vector divided by its $z$ component and scaled by $\color{green}{H(A)}$. We also introduced a `height_scale` uniform for some extra control as the parallax effect is usually too strong without an extra scale parameter. We then subtract this vector $\color{brown}{\bar{P}}$ from the texture coordinates to get the final displaced texture coordinates. 

What is interesting to note here is the division of `viewDir.xy` by `viewDir.z`. As the `viewDir` vector is normalized, `viewDir.z` will be somewhere in the range between $0.0$ and $1.0$. When  `viewDir` is largely parallel to the surface, its z component is close to $0.0$ and the division returns a much larger vector $\color{brown}{\bar{P}}$ compared to when `viewDir` is largely perpendicular to the surface. We're adjusting the size of $\color{brown}{\bar{P}}$ in such a way that if offsets the texture coordinates at a larger  




















