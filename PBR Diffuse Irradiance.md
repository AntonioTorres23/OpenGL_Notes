
IBL, or **image based lighting**, is a collection of techniques to light objects, not by direct analytical lights as in the [previous](https://learnopengl.com/PBR/Lighting) notes, but by treating the surrounding environment as one big light source. This is generally accomplished by manipulating a cubemap environment map (taken from the real world or generated from a 3D scene) such that we can directly use it in our lighting equations: treating each cubemap texel as a light emitter. This way we can effectively capture an environment's global lighting and general feel, giving objects a better sense of *belonging* in their environment. 

As image based lighting algorithms capture the lighting of some (global) environment, its input is considered a more precise form of ambient lighting, even a crude approximation of global illumination. This makes IBL interesting for PBR as objects look significantly more physically accurate when we take the environments lighting into account. 

To start introducing IBL into our system let's take a quick look at the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$    

As described before, our main goal is to solve the integral of all incoming light directions $\Large{\omega_i}$ over the hemisphere $\Large{\Omega}$. Solving the integral in the previous notes was easy as we knew beforehand the exact few light directions $\Large{\omega_i}$ that contributed to the integral. This time however, **every** incoming light direction $\Large{\omega_i}$ from the surrounding environment could potentially have some radiance making it less trivial to solve the integral. This gives us two main requirements for solving the integral. 

- We need some way to retrieve the scene's radiance given any direction vector $\Large{\omega_i}$. 
- Solving the integral needs to be fast and real-time

Now, the first requirement is relatively easy. We've already hinted it, but one way of representing an environment or scene's irradiance is in the form of a (processed) environment cubemap. Given such a cubemap, we can visualize every texel of the cubemap as one single emitting light source. By sampling this cubemap with any direction vector $\Large{\omega_i}$, we retrieve the scene's radiance from that direction. 

Getting the scene's radiance given any direction vector $\Large{\omega_i}$ is then as simple as. 

`vec3 radiance = texture(_cubemapEnvironment, w_i).rgb;`

Still, solving the integral requires us to sample the environment map from not just one direction, but all possible directions $\Large{\omega_i}$ over the hemisphere $\Large{\Omega}$ which is far too expensive for each fragment shader invocation. To solve the integral in a more efficient fashion we'll want to *pre-process* or **pre-compute** most of the computations. For this we'll have to delve a bit deeper into the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$ 

Taking a good look at the reflectance equation we find that the diffuse $\Large{k_d}$ and specular $\Large{k_s}$ term of the BRDF are independent from each other and we can split the integral in two. 


$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi}) L_i(p, \omega_i)n \cdot \omega_i d \omega_i + \int\limits_{\Omega} (\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$ 

By splitting the integral in two parts we can focus on both the diffuse and specular term individually; the focus of these notes being on the diffuse integral. 

Taking a closer look at the diffuse integral we find that the diffuse lambert term is a constant term (the color **c**, the refraction ratio $\Large{k_d}$, and $\Large{\pi}$ are constant over the integral) and not dependent on any of the integral variables. Given this, we can move the constant term out of the diffuse integral. 

$\LARGE{L_o(p, \omega_o) = k_d \frac{c}{\pi} \int\limits_{\Omega} L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$ 
This gives us an integral that only depends on $\Large{\omega_i}$ (assuming $\Large{p}$ is at the center of the environment map). With this knowledge, we can calculate or *pre-compute* a new cubemap that stores in each sample direction (or texel) $\Large{\omega_o}$ the diffuse integral's result by **convolution**.

Convolution is apply some computation to each entry in a data set considering all other entries in the data set; the data set being the scene's radiance or environment map. Thus for every sample direction in the cubemap, we take all other sample directions over the hemisphere $\Large{\Omega}$ and averaging their radiance. The hemisphere we build the sample directions $\Large{\omega_i}$ from is oriented towards the output $\Large{\omega_o}$ sample direction we're convoluting. 

![[Pasted image 20251215123144.png]]

This pre-computed cubemap, that for each sample direction $\Large{\omega_o}$ stores the integral result, can be though of as the pre-computed sum of all indirect diffuse light of the scene hitting some surface aligned along direction $\Large{\omega_o}$. Such a cubemap is known as an **irradiance map** seeing as the convoluted effectively allows us to directly sample the scene's (pre-computed) irradiance from any direction $\Large{\omega_o}$. 

The radiance equation also depends on a position $\Large{p}$, which we've assumed to be at the center of the irradiance map. This does mean all diffuse indirect light must come from a single environment map which may break the illusion of reality (especially indoors). Render engines solve this by placing **reflection probes** all over the scene where each reflection probes calculates its own irradiance map of its surroundings. This way, the irradiance (and radiance) at position $\Large{p}$ is the interpolated irradiance between its closest reflection probes. For now, we assume we always sample the environment map from its center. 

Below is an example of a cubemap environment map and its resulting irradiance map (courtesy of [wave engine](http://www.indiedb.com/features/using-image-based-lighting-ibl)), averaging the scene's radiance for every direction $\Large{\omega_o}$. 

![[Pasted image 20251215124413.png]]

By storing the convoluted result in each cubemap texel (in the direction of $\Large{\omega_o}$), the irradiance map displays somewhat like an average color or lighting display of the environment. Sampling any direction from this environment map will give us the scene's irradiance in that particular direction. 

**PBR and HDR**

We've briefly touched upon it in the [previous](https://learnopengl.com/PBR/Lighting) notes: taking the high dynamic range of your scene's lighting into account in a PBR pipeline is incredibly important. As PBR bases most of its inputs on real physical properties and measurements it makes sense to closely match the incoming light values to their physical equivalents. Whether we make educated guesses on each light's radiant flux or use their [direct physical equivalent](https://en.wikipedia.org/wiki/Lumen_\(unit\)), the difference between a simple light bulb or the sun is significant either way. Without working in an [HDR](https://learnopengl.com/Advanced-Lighting/HDR) render environment it's impossible to correctly specify each light's relative intensity. 

So, PBR and HDR go hand in hand, but how does it relate to image based lighting? We've seen in the previous notes that it's relatively easy to get PBR working in HDR. However, seeing as for image based lighting we base the environment's indirect light intensity on the color values of an environment cubemap we need some way to store the lighting's high dynamic range into an environment map. 

The environment maps we've been using so far as cubemaps (used as [skyboxes](https://learnopengl.com/Advanced-OpenGL/Cubemaps) for instance) are in low dynamic range (LDR). We directly used their color values from the individual face images, ranged between $0.0$ and $1.0$, and processed them as is. While this may work fine for visual output, when taking them as physical input parameters it's not going to work. 

**The Radiance HDR File Format**

Enter the radiance file format. The radiance file format (with the `.hdr` extension) stores a full cubemap with all 6 faces as floating point data. This allows us to specify color values outside the $0.0$ and $1.0$ range to give lights their correct color intensities. The file format also uses a clever trick to store each floating point value, not as a 32 bit value per channel, but 8 bits per channel using the color's alpha channel as an exponent (this does come with a loss of precision). This works quite well, but requires the parsing program to re-convert each color to their floating point equivalent.

There are quite a few radiance HDR environment maps freely available form sources like [sIBL archive](http://www.hdrlabs.com/sibl/archive.html) of which you can see an example below. 

![[Pasted image 20251215130835.png]]

This may not be exactly what you were expecting, as the image appears distorted and doesn't show any of the 6 individual cubemap faces of environment maps we've seen before. This environment map is projected from a sphere onto a flat plane such that we can more easily store the environment into a single image known as an **equirectangular map**. This does come with a small caveat as most of the visual resolution is stored in the horizontal view direction, while less is preserved in the bottom and top directions. In most cases this is a decent compromise as with almost any renderer you'll find most of the interesting lighting and surroundings in the horizontal viewing directions. 

**HDR and stb_image.h**

Loading radiance HDR images directly requires some knowledge of the [file format](http://radsite.lbl.gov/radiance/refer/Notes/picture_format.html) which isn't too difficult, but cumbersome nonetheless. Lucky for us, the popular one header library [stb_image.h](https://github.com/nothings/stb/blob/master/stb_image.h) supports loading radiance HDR images directly as an array of floating point values which perfectly fits our needs. With `stb_image` added to your project, loading an HDR image is now as simple as follows. 

```
#include "stb_image.h"

[...]

stbi_set_flip_vertically_on_load(true);
in width, height, nrComponents; 
float *data = stbi_loadf("newport_loft.hdr", &width, &height, &nrComponents, 0);
unigned int hdrTexture;
if (data)
{
	glGenTextures(1, &hdrTexture);
	glBindTexture(GL_TEXTURE_2D, hdrTexture);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, width, height, 0, GL_RGB, GL_FLOAT,      data);
	
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	
	stbi_image_free(data);
}
else
{
	std::cout << "Failed to load HDR image." << std::endl;
}
```

`stb_image.h` automatically maps the HDR values to a list of floating point values: 32 bits per channel and 3 channels per color by default. This is all we need to store the equirectangular HDR environment map into a 2D floating point texture. 

**From Equirectangular to Cubemap**

It is possible to use the equirectangular map directly for environment lookups, but these operations can be relatively expensive in which case a direct cubemap sample is more performant. Therefore, in these notes we'll first convert the equirectangular image to a cubemap for further processing. Note that in the process we also show how to sample an equirectangular map as if it was a 3D environment map in which case you're free to pick whichever solution you prefer. 

To convert an equirectangular image into a cubemap we need to render a (unit) cube and project the equirectangular map on all of the cube's faces from the inside and take 6 images of each of the cube's sides as a cubemap face. The vertex shader of this cube simply renders the cube as is and passes its local position to the fragment shader as a 3D sample vector. 

```
#version 330 core
layout (location = 0) in vec3 aPos; 

out vec3 localPos;

uniform mat4 projection;
uniform mat4 view; 

void main()
{
	localPos = aPos;
	gl_Position = projection * view * vec4(localPos, 1.0);
}
```

For the fragment shader, we color each part of the cube as if we neatly folded the equirectangular map onto each side of the cube. To accomplish this, we take the fragment's sample direction as interpolated from the cube's local position and then use this direction vector and some trigonometry magic (spherical to cartesian) to sample the equirectangular map as if it's a cubemap itself. We directly store the result onto the cube-face's fragment which should be all we need to do. 

```
#version 330 core
out vec4 FragColor; 
in vec3 localPos; 

uniform sampler2D equirectangularMAP;

const vec2 invAtan = vec2(0.1591, 0.3183);
vec2 SampleSphericalMap(vec3 v)
{
	vec2 uv = vec2(atan(v.z, v.x), asin(v.y));
	uv *= invAtan;
	uv += 0.5;
	return uv;
}

void main()
{
	// make sure to normalize localPos; 
	vec2 uv = SampleSphericalMap(normalize(localPos)); 
	vec3 color = texture(equirectangularMap, uv).rgb;
	
	FragColor = vec4(color, 1.0);
}
```

If you render a cube at the center of the scene given an HDR equirectangular map you'll get something that looks like this. 

![[Pasted image 20251215144028.png]]

This demonstrates that we effectively mapped an equirectangular image onto a cubic shape, but doesn't yet help us in converting the source HDR image to a cubemap texture. To accomplish this we have to render the same cube 6 times, looking at each individual face of the cube while recording its visual result with a [framebuffer](https://learnopengl.com/Advanced-OpenGL/Framebuffers) object. 

```
unsigned int captureFBO, captureRBO;
glGenFramebuffers(1, &captureFBO);
glGenRenderbuffers(1, &captureRBO);

glBindFramebuffer(GL_FRAMEBUFFER, captureFBO);
glBindRenderbuffer(GL_RENDERBUFFER, captureRBO);
glRenderBufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24, 512, 512);
glFramebufferRenderBuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, captureRBO);
```

Of course, we then also generate the corresponding cubemap textures, pre-allocating memory for each of its 6 faces. 

```
unsigned int envCubemap;
glGenTextures(1, &envCubemap);
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);

for(int i = 0; i < 6; ++i)
{
	// note that we store each face with 16 bit floating point values
	glTexImage2D(GL_TEXTURE_CUPMAP_POSITIVE_X + i, 0, GL_RGB16F, 512, 512, 0,          GL_RGB, GL_FLOAT, nullptr);
}

glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

Then what's left to do is capture the equirectangular 2D texture onto cubemap faces. 

I won't go over the details as the code details topics previously discussed in the [framebuffer](https://learnopengl.com/Advanced-OpenGL/Framebuffers) and [point shadows](https://learnopengl.com/Advanced-Lighting/Shadows/Point-Shadows) notes, but it effectively boils down to setting up 6 different view matrices (facing each side of the cube), set up a projection matrix with a fov of 90 degrees to capture the entire face, and render a cube 6 times storing the results in a floating point framebuffer .

```
glm::mat4 captureProjection = glm::perspective(glm::radians(90.0f), 1.0f, 0.1f, 10.0f);
glm::mat4 captureViews[] = 
{
	glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(1.0f, 0.0f, 0.0f),              glm::vec3(0.0f, -1.0f, 0.0f)),
	glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(-1.0f, 0.0f, 0.0f),             glm::vec3(0.0f, -1.0f, 0.0f));
	glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f),              glm::vec3(0.0f, 0.0f, 1.0f));
	glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, -1.0f, 0.0f),             glm::vec3(0.0f, 0.0f, -1.0f));
	glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 0.0f, 1.0f),              glm::vec3(0.0f, -1.0f, 0.0f));
	glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm:vec3(0.0f, 0.0f, -1.0f),              glm::vec3(0.0f, -1.0f, 0.0f));
}

// convert HDR equirectangular environment map to cubemap equivalent
equirectangularToCubemapShader.use();
equirectangularToCubemapShader.setInt("equirectangularMap", 0);
equirectangularToCubemapShader.setMat4("projection", captureProjection);
glActivateTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, hdrTexture);

// don't forget to configure the viewport to the capture dimensions
glViewport(0, 0, 512, 512); 
glBindFramebuffer(GL_FRAMEBUFFER, captureFBO);
for (unsigned int i = 0; i < 6; ++i)
{
	equirectangularToShader.setMat4("view", captureViews[i]);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,                       GL_COLOR_ATTACHMENT0, GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, envCubemap, 0);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	
	renderCube(); // renders a 1x1 cube
	
}
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

We take the color attachment of the framebuffer and switch its texture around for every face of the cubemap, directly rendering the scene into one of the cubemap's faces. Once this routine has finished (which we only do once), the cubemap `envCubemap` should be the cubemapped environment version of our original HDR image. 

Let's test the cubemap by writing a very simple skybox shader to display the cubemap around us. 

```
#version 330 core
layout(location = 0) in vec3 aPos; 

uniform mat4 projection; 
uniform mat4 view; 

out vec3 localPos; 

void main()
{
	localPos = aPos;
	
	mat4 rotView = mat4(mat3(view)); // remove translation from the view matrix
	mat4 clipPos = projection * rotView * vec4(localPos, 1.0);
	
	// perspective division makes the z and w componet equal to 1 since we replace 
	// w as our z value
	gl_Position = clipPos.xyww; 
}
```

Note the `xyww` trick here that ensures the depth value of the rendered cube fragments always end up at $1.0$, the maximum depth value, as described in the [cubemap](https://learnopengl.com/Advanced-OpenGL/Cubemaps) notes. Do note that we need to change the depth comparison to `GL_LEQUAL`. 

`glDepthFunc(GL_LEQUAL);`

The fragment shader then directly samples the cubemap environment map using the cube's local fragment position. 

```
#version 330 core
out vec4 FragColor;

in vec3 localPos; 

uniform samplerCube environmentMap;

void main()
{
	vec3 envColor = texture(environmentMap, localPos).rgb;
	
	envColor = envColor / (envColor + vec3(1.0));
	envColor = pow(envColor, vec3(1.0/2.2));
	
	FragColor = vec4(envColor, 1.0);
}
```


We sample the environment map using its interpolated vertex cube positions that directly correspond to the correct direction vector to sample. Seeing as the camera's translation components are ignored, rendering this shader over a cube should give you the environment map as a non-moving background. Also, as we directly output the environment map's HDR values to the default LDR framebuffer, we want to properly tone map the color values. Furthermore, almost all HDR maps are in linear color space by default so we need to apply [gamma correction](https://learnopengl.com/Advanced-Lighting/Gamma-Correction) before writing to the default framebuffer. 

Now rendering the sampled environment map over the previously rendered spheres should look something like this. 

![[Pasted image 20251215162242.png]]

Well... it took us quite a bit of setup to get here, but we successfully managed to read an HDR environment map, convert it from its equirectangular mapping to a cubemap, and render the HDR cubemap into the scene as a skybox. Furthermore, we set up a small system to render onto all 6 faces of a cubemap, which we'll need again when **convoluting** the environment map. You can find the source code of the entire conversion process [here](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.1.1.ibl_irradiance_conversion/ibl_irradiance_conversion.cpp).

**Cubemap Convolution**

As described at the start of the notes, our main goal is to solve the integral for all diffuse indirect lighting given the scene's irradiance in the form of a cubemap environment map. We know that we can get the radiance of the scene $\Large{L(p, \omega_i)}$ in a particular direction by sampling an HDR environment map in direction $\Large{\omega_i}$. To solve the integral, we have to sample the scene's radiance from all possible directions within the hemisphere $\Large{\Omega}$ for each fragment. 

It is however computationally impossible to sample the environment's lighting from every possible direction in $\Large{\Omega}$, the number of possible directions is theoretically infinite. We can however, approximate the number of directions by taking a finite number of directions or samples, spaced uniformly or taken randomly from within the hemisphere, to get fairly accurate approximation of the irradiance; effectively solving the integral $\Large{\int}$ discretely. 

It is however still too expensive to do this for every fragment in real-time as the number of samples needs to be significantly large for decent results, so we want to **pre-compute** this. Since the orientation of the hemisphere decides where we capture the irradiance, we can pre-calculate the irradiance for every possible hemisphere orientation oriented around all outgoing directions $\Large{\omega_o}$

$\LARGE{L_o(p, \omega_o) = k_d \frac{c}{\pi} \int\limits_{\Omega} L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$

Given any direction vector $\Large{\omega_i}$ in the lighting pass, we can then sample the pre-computed irradiance map to retrieve the total diffuse irradiance from direction $\Large{\omega_i}$. To determine the amount of indirect diffuse (irradiant) light at a fragment surface, we retrieve the total irradiance from the hemisphere oriented around its surface normal. Obtaining the scene's irradiance is then as simple as. 

`vec3 irradiance = texture(irradianceMap, N).rgb;`

Now, to generate the irradiance map, we need to convolute the environment's lighting as converted to a cubemap. Given that for each fragment the surface's hemisphere is oriented along the normal vector $\Large{N}$, convoluting a cubemap equals the total average radiance of each direction $\Large{\omega_i}$ in the hemisphere $\Large{\Omega}$ oriented along $\Large{N}$. 

![[Pasted image 20251215164303.png]]

Thankfully, all the cumbersome setup of these notes isn't all for nothing as we can now directly take the converted cubemap, convolute it in a fragment shader, and capture its result in a new cubemap using a framebuffer that render to all 6 face directions. As we've already set this up for converting the equirectangular environment map to a cubemap, we can take the exact same approach but use a different fragment shader. 

```
#version 330 core
out vec4 FragColor; 
in vec3 localPos; 

uniform samplerCube environmentMap;

const float PI = 3.14159265359;

void main()
{
	// the sample direction equals the hemisphere's orientation
	vec3 normal = normalize(localPos);
	
	vec3 irradiance = vec3(0.0);
	
	[...] // convolution code
	
	FragColor = vec4(irradiance, 1.0);
}
```
 
 With `environmentMap` being the HDR cubemap as converted from the equirectangular HDR environment map. 

There are many ways to convolute the environment map, but for these notes we're going to generate a fixed amount of sample vectors for each cubemap texel along a hemisphere $\Large{\Omega}$ oriented around the sample direction and average the results. The fixed amount of sample vectors will be uniformly spread inside the hemisphere. Note that an integral is a continuous function and discretely sampling its function given a fixed amount of sample vectors will be an approximation. The more sample vectors we use, the better we approximate the integral. 

The integral $\Large{\int}$ of the reflectance equation resolves around the solid angle $\Large{d \omega}$ which is rather difficult to work with. Instead of integrating over the solid angle $\Large{d \omega}$ we'll integrate over its equivalent spherical coordinates $\Large{\theta}$ and $\Large{\phi}$. 

![[Pasted image 20251216102028.png]]
We use the polar azimuth $\Large{\phi}$ angle to sample around the ring of the hemisphere between $0$ and $\Large{2 \pi}$, and use the inclination zenith $\Large{\theta}$ angle between $0$ and $\Large{\frac{1}{2} \pi}$ to sample the increasing rings of the hemisphere. This will give us the updated reflectance integral. 

$\LARGE{L_o(p, \phi_o, \theta_o) = k_d \frac{c}{\pi} \int_{\phi = 0}^{2 \pi} \int_{\theta = 0}^{\frac{1}{2}\pi} L_i(p, \phi_i, \theta_i) \cos(\theta) \sin(\theta) d \phi d \theta}$    

So I think the polar azimuth measures around the ring of the hemisphere and the inclination zenith measures up and down the hemisphere. Essentially creating a ring above or below an already existing one. Just think of polar as polarity and inclination as incline meaning up or down. 

Solving the integral requires us to take a fixed number of discrete samples within the hemisphere $\Large{\Omega}$ and averaging their results. This translates the integral to the following discrete version as based on the [Riemann sum](https://en.wikipedia.org/wiki/Riemann_sum) given $\Large{n1}$ and $\Large{n2}$ discrete samples on each spherical coordinate respectively. 

$\LARGE{L_o(p, \phi_o, \theta_o) = k_d \frac{c \pi}{n1 n2} \sum_{\phi = 0}^{n1} \sum_{\theta = 0}^{n2} L_i(p, \phi_i, \theta_i) \cos(0) \sin(\theta) d \phi d \theta}$        

As we sample both spherical values discretely, each sample will approximate or average an area on the hemisphere as the image before shows. Note that (due to the general properties of a spherical shape) the hemisphere's discrete sample area gets smaller the higher the zenith angle $\Large{\theta}$ as the sample regions converge towards the center top. To compensate for the smaller areas, we weigh its contribution by scaling the area by $\Large{\sin \theta}$. 

Discretely sampling the hemisphere given the integral's spherical coordinates translates to the following fragment code. 

```
vec3 irradiance = vec3(0.0);

vec3 up    = vec3(0.0, 1.0, 0.0);
vec3 right = normalize(cross(up, normal));
up         = normalize(cross(normal, right));

float sampleDelta = 0.025;
float nrSamples = 0.0;
for(float phi = 0.0; phi < 2.0 * PI; phi 
```



























