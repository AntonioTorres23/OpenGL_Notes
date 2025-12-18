
In the [previous](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) notes we've set up PBR in combination with image based lighting by pre-computing an irradiance map as the lighting's indirect diffuse portion. In these notes we'll focus on the specular part of the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + k_s \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$

You'll notice that the Cook-Torrance specular portion (multiplied by $\Large{kS}$) isn't constant over the integral and is dependent on the incoming light direction, but **also** the incoming view direction. Trying to solve the integral for all incoming light including all possible view directions is a combinatorial overload and way too expensive to calculate on a real-time basis. Epic Games proposed a solution where they were able to pre-convolute the specular part for real-time purposes, given a few compromises, known as the **split sum approximation**. 

The split sum approximation splits the specular part of the reflectance equation into two separate parts that we can individually convolute and later combine in the PBR shader for specular indirect image based lighting. Similar to how we pre-convoluted the irradiance map, the split sum approximation requires an HDR environment map as its convolution input. To understand the split sum approximation we'll again look at the reflectance equation, but this time focus on the specular part. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} k_s \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i} = \int\limits_{\Omega} f_r (p, \omega_i, \omega_o) L_i (p, \omega_i) n \cdot \omega_i d \omega_i$  
For the same (performance) reasons as the irradiance convolution, we can't solve the specular part of the integral in real time and expect a reasonable performance. So preferably we'd per-compute this integral to get something like a specular IBL map, sample this map with the fragment's normal, and be done with it. However, this is where it gets a bit tricky. We were able to pre-compute the irradiance map as the integral only depended on $\Large{\omega_i}$ and we could move the constant diffuse albedo terms out of the integral. This time, the integral depends on more than just $\Large{\omega_i}$ as evident from the BRDF. 

$\LARGE{f_r (p, \omega_i, \omega_o) = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}}$ 

The integral also depends on $\Large{\omega_o}$, and we can't really sample a pre-computed cubemap with two direction vectors. The position $\Large{p}$ is irrelevant here as described in the previous notes. Pre-computing this integral for every possible combination of $\Large{\omega_i}$ and $\Large{\omega_o}$ isn't practical in a real-time setting. 

Epic Games' split sum approximation solves the issue by splitting the pre-computation into 2 individual parts that we can later combine to get the resulting pre-computed result we're after. The split-sum approximation splits the specular integral into two separate integrals. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega}L_i(p, \omega_i)d \omega_i * \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d \omega_i}$   

The first part (when convoluted) is known as the **pre-filtered environment map** which is (similar to the irradiance map) a pre-computed environment convolution map, but this time taking roughness into account. For increasing roughness levels, the environment is convoluted with more scattered sample vectors, creating blurrier reflections. For each roughness level we convolute, we store the sequentially blurrier results in the pre-filtered map's mipmap levels. For instance, a pre-filtered environment map storing the pre-convoluted result of 5 different roughness values in its 5 mipmap levels looks as follows. 

![[Pasted image 20251217110922.png]]

We generate the sample vectors and their scattering amount using the normal distribution function (NDF) of the Cook-Torrance BRDF that takes as input both a normal and view direction. As we don't know beforehand the view direction when convoluting the environment map, Epic Games makes a further approximation by assuming the view direction (and thus the specular reflection direction) to be equal to the output sample direction $\Large{\omega_o}$. This translates itself to the following code. 

```
vec3 N = normalize(w_o);
vec3 R = N;
vec3 V = R;
```

This way, the pre-filtered environment convolution doesn't need to be aware of the view direction. This does mean we don't get nice grazing specular reflections when looking at specular surface reflections from an angle as seen in the image below (courtesy of *Moving Frostbite to PBR* article); this is however generally considered an acceptable compromise. 

![[Pasted image 20251217112313.png]]


The second part of the split sum equation equals the BRDF part of the specular integral. If we pretend the incoming radiance is completely white for every direction (thus $\Large{L(p, x) = 1.0}$) we can pre-calculate the BRDF's response given an input roughness and an input angle between the normal $\Large{n}$ and the light direction $\Large{\omega_i}$, or $\Large{n \cdot \omega_i}$. Epic Games stores the pre-computed BRDF's response to each normal and light direction combination on varying roughness values in a 2D lookup texture (LUT) known as the **BRDF integration** map. The 2D lookup texture outputs a scale (red) and a bias value green (green) to the surface's Fresnel response giving us the second part of the split specular integral. 

![[Pasted image 20251217122056.png]]

We generate the lookup texture by treating the horizontal texture coordinate (ranged between $0.0$ and $1.0$) of a plane as the BRDF's input $\Large{n \cdot \omega_i}$, and its vertical texture coordinate as the input roughness value. With this BRDF integration map and the pre-filtered environment map we can combine both to get the result of the specular integral. 

```
float lod             = getMipLevelFromRoughness(roughness);
vec3 prefilteredColor = textureCubeLod(PrefilteredEnvMap, refVec, lod);
vec2 envBRDF          = texture2D(BRDFIntegrationMap, vec2(NdotV, roughness)).xy;
vec3 indirectSpecular = prefilteredColor * (F * envBRDF.x + envBRDF.y);
```

This should give you a bit of an overview on how Epic Games' split sum approximation roughly approaches the indirect part of the reflectance equation. Let's now try and build the pre-convoluted parts ourselves. 

**Pre-filtering an HDR Environment Map**

Pre-filtering an environment map is quite similar to how we convoluted an irradiance map. The difference being that we now account for roughness and store sequentially rougher reflections in the pre-filtered map's mip levels. 

First, we need to generate a new cubemap to hold the pre-filtered environment data. To make sure we allocate enough memory for its mip levels we call `glGenerateMipmap` as an easy way to allocate the required amount of memory. 

```
unsigned int prefilterMap;
glGenTextures(1, &prefilterMap);
glBindTexture(GL_TEXTURE_CUBE_MAP, prefilterMap);
for (unsigned int i = 0; i < 6; ++i)
{
	glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGB16F, 128, 128, 0,        GL_RGB, GL_FLOAT, nullptr);
}
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

glGenerateMipmap(GL_TEXTURE_CUBE_MAP);
```

Note that because we plan to sample `prefilterMap`'s mipmaps you'll need to make sure its minification filter is set to `GL_LINEAR_MIPMAP_LINEAR` to enable trilinear filtering. We store the pre-filtered specular reflections in a per-face resolution of 128 by 128 at its base mip level. This is likely enough for most reflections, but if you have a large number of smooth materials (think of car reflections) you may want to increase the resolution. 

In the previous notes we convoluted the environment map by generating sample vectors uniformly spread over the hemisphere $\Large{\Omega}$ using spherical coordinates. While this works just fine for irradiance, for specular reflections its less efficient. When it comes to specular reflections, based on the roughness of a surface, the light reflects closely or roughly around a reflection vector $\Large{r}$ over a normal $\Large{n}$, but (unless the surface is extremely rough) around the reflection vector nonetheless.

![[Pasted image 20251217133939.png]]

The general shape of possible outgoing light reflections is known as the **specular lobe**. As roughness increases, the specular lobe's size increases; and the shape of the specular lobe changes on varying incoming light directions. The shape of the specular lobe is thus highly dependent on that material. 

When it comes to the microsurface model, we can imagine the specular lobe as the reflection orientation about the microfacet halfway vectors given some incoming light direction. Seeing as most light rays end up in a specular lobe reflected around the microfacet halfway vectors, it makes sense to generate the sample vectors in a similar fashion as most would otherwise be wasted. This process is known as **importance sampling**.

**Monte Carlo Integration and Importance Sampling**

To fully get a grasp of importance sampling it's relevant we first delve into the mathematical construct known as **Monte Carlo Integration**. Monte Carlo integration revolves mostly around a combination of statistics and probability theory. Monte Carlo helps us in discretely solving the problem of figuring out some statistic or value of a population without having to take **all** of the population into consideration. 

For instance, let's say you want to count the average height of all citizens of a country. To get your result, you could measure **every** citizen and average their height which will give you the **exact** answer you are looking for. However, since most countries have a considerable population this isn't a realistic approach: it would take too much effort and time. 

A different approach is to pick a much smaller **completely random** (unbiased) subset of this population, measure their height, and average the result. This population could be as small as 100 people. While not as accurate as the exact answer, you'll get an answer that is relatively close to the ground truth. This is known as the **law of large numbers**. The idea is that if you measure a smaller set of size $\Large{N}$ of truly random samples from the population, the result will be relatively close to the true answer and gets closer as the number of samples $\Large{N}$ increases. 

Monte Carlo integration builds on this law of large numbers and takes the same approach in solving an integral. Rather than solving an integral for all possible (theoretically infinite) sample values $\Large{x}$, simply generate $\Large{N}$ sample values randomly picked from the total population and average. As $\Large{N}$ increases, we're guaranteed to get a result closer to the exact answer of the integral. 

$\LARGE{O = \int\limits_{a}^{b} f(x) d x = \frac{1}{N} \sum_{i = 0}^{N - 1} \frac{f(x)}{pdf(x)}}$  

To solve the integral, we take $\Large{N}$ random samples over the population $\Large{a}$ to $\Large{b}$, add them together, and divide by the total number of samples to average them. The $\Large{pdf}$ stands for the **probability density function** that tells us the probability a specific sample occurs over the total sample set. For instance, the pdf of the height of a population would look a bit like this. 

![[Pasted image 20251217141255.png]]

From this graph we can see that if we take any random sample of the population, there is a higher chance of picking a sample of someone of height 1.70, compared to the lower probability of the sample being of height 1.50. 

When it comes to Monte Carlo integration, some samples may have a higher probability of being generated than others. This is why for any general Monte Carlo estimation we divide or multiply the sampled value by the sample probability according to a pdf. So far, in each of our cases of estimating an integral the samples we've generated were uniform, having the exact same change of being generated. Our estimations so far were **unbiased**, meaning that given an ever-increasing amount of samples we will eventually **converge** to the **exact** solution of the integral. 

However, some Monte Carlo estimators are **biased**, meaning that the generated samples aren't completely random, but focused towards a specific value or direction. These biased Monte Carlo estimators have a **faster rate of convergence**, meaning they can converge to the exact solution at a much faster rate, but due to their biased nature it's likely they won't ever converge to the exact solution. This is generally an acceptable tradeoff, especially in computer graphics, as the exact solution isn't too important as long as the results are visually acceptable. As we'll soon see with importance sampling (which uses a biased estimator), the generated samples are biased towards specific directions in which case we account for this by multiplying or dividing each sample by its corresponding pdf. 

Monte Carlo integration is quite prevalent in computer graphics as it's a fairly intuitive way to approximate continuous integrals in a discrete and efficient fashion: take any area/volume to sample over (like the hemisphere $\Large{\Omega}$), generate $\Large{N}$ amount of random samples within the area/volume, and sum and weigh every sample contribution to the final result. 

Monte Carlo integration is an extensive mathematical topic and I won't delve much further into specifics, but we'll mention that there are multiple ways of generating the *random samples*. By default, each sample is completely (pseudo) random as we're used to, but by utilizing certain properties of somewhat-random sequences we can generate sample vectors that are still random, but have interesting properties. For instance, we can do Monte Carlo integration on something called **low-discrepancy sequences** which still generate random samples, but each sample is more evenly distributed (image courtesy of James Heald).

![[Pasted image 20251217143747.png]]
  

When using a low-discrepancy sequence for generating the Monte Carlo sample vectors, the process is known as **Quasi-Monte Carlo Integration**. Quasi-Monte Carlo methods have a faster **rate of convergence** which makes them interesting for performance heavy applications. 

Given our newly obtained knowledge of Monte Carlo and Quasi-Monte Carlo integration, there is an interesting property we can use for an even faster rate of convergence known as **importance sampling**. We've mentioned it before in these notes, but when it comes to specular reflections of light, the reflected light vectors are constrained in a specular lobe with its size determined by the roughness of the surface. Seeing as any (quasi-) randomly generated sample outside the specular lobe isn't relevant to the specular integral it makes sense to focus the sample generation to within the specular lobe, at the cost of making the Monte Carlo estimator biased.

This is in essence what importance sampling is about: generate sample vectors in some region constrained by roughness oriented around the microfacet's halfway vector. By combining Quasi-Monte Carlo sampling with a low-discrepancy sequence and biasing the sample vectors using importance sampling, we get a high rate of convergence. Because we reach the solution at a faster rate, we'll need significantly fewer samples to reach an approximation that is sufficient enough. 

**A Low-Discrepancy Sequence**

In these notes we'll pre-compute the specular portion of the indirect reflectance equation using importance sampling given a random low-discrepancy sequence based on the Quasi-Monte Carlo method. The sequence we'll be using is known as the **Hammersley Sequence** as carefully described by [Holger Dammertz](http://holger.dammertz.org/stuff/notes_HammersleyOnHemisphere.html). The Hammersley sequence is based on the **Van Der Corput** sequence which mirrors a decimal binary representation around its decimal point. 

Given some neat bit tricks, we can quite efficiently generate the Van Der Corput sequence in a shader program which we'll use to get a Hammersley sequence sample `i` over N total samples. 

```
float RadicalInverse_VdC(uint bits)
{
	bits = (bits << 16u) | (bits >> 16u);
	bits = ((bits & 0x55555555u) << 1u) | ((bits & 0xAAAAAAAAu) >> 1u);
	bits = ((bits & 0x33333333u) << 2u) | ((bits & 0xCCCCCCCCu) >> 2u);
	bits = ((bits & 0x0F0F0F0Fu) << 4u) | ((bits & 0xF0F0F0F0u) >> 4u);
	bits = ((bits & 0x00FF00FFu) << 8u) | ((bits & 0xFF00FF00u) >> 8u);
	return float(bits) * 2.3283064365386963e-10; // / 0x100000000
}

vec2 Hammersley(uint i, uint N)
{
	return vec2(float(i)/float(N), RadicalInverse_VdC(i));
}
```

The GLSL Hammersley function gives us the low-discrepancy sample `i` of the total sample set of size `N`. 

**Hammersley Sequence Without Bit Operator Support**

Not all OpenGL related drivers support bit operators (WebGL and OpenGL ES 2.0 for instance) in which case you may want to use an alternative version of the Van Der Corput Sequence that doesn't rely on bit operators. 

```
float VanDerCorput(uint n, uint base)
{
	float invBase = 1.0 / float(base);
	float denom   = 1.0;
	float result  = 0.0;
	
	for(uint i = 0u; i < 32u, ++i)
	{
		if (n > 0u)
		{
			denom   = mod(float(n), 2.0);
			result += denom * invBase;
			invBase = invBase / 2.0;
			n       = uint(float(n) / 2.0); 
		}
	}
	return result;
}

vec2 HammersleyNoBitOps(uint i, uint N)
{
	return vec2(float(i)/float(N), VanDerCorput(i, 2u));
}
```

Note that due to GLSL loop restrictions in older hardware, the sequence loops over all 32 bits. This version is less performant, but does work on all hardware if you ever find yourself without bit operators.

**GGX Importance Sampling**

Instead of uniformly or randomly (Monte Carlo) generating sample vectors over the integral's hemisphere $\Large{\Omega}$, we'll generate sample vectors biased towards the general reflection orientation of the microsurface halfway vector based on the surface's roughness. The sampling process will be similar to what we've seen before: begin a large loop, generate a random (low-discrepancy) sequence value, take the sequence value to generate a sample vector in tangent space, transform to world space, and sample the scene's radiance. What's different is that we now use a low-discrepancy sequence value as input to generate a sample vector. 

```
const uint SAMPLE_COUNT = 4096u;
for(uint i = 0u; i < SAMPLE_COUNT; ++i)
{
	vec2 Xi = Hammersley(i, SAMPLE_COUNT);
}
```

Additionally, to build a sample vector, we need some way of orienting and biasing the sample vector towards the specular lobe of some surface roughness. We can take the NDF as described in the [theory](https://learnopengl.com/PBR/Theory) notes and combine the GGX NDF in the spherical sample vector process as described by Epic Games. 

```
vec3 ImportanceSampleGGX(vec2 Xi, vec3 N, float roughness)
{
	float a = roughness * roughness; 
	
	float phi = 2.0 * PI * Xi.x;
	float cosTheta = sqrt((1.0 - Xi.y) / (1.0 + (a*a - 1.0) * Xi.y));
	float sinTheta = sqrt(1.0 - cosTheta*cosTheta);
	
	// from sphereical coordinates to cartesian coordaintes
	vec3 H;
	H.x = cos(phi) * sinTheta;
	H.y = sin(phi) * sinTheta;
	H.z = cosTheta;
	
	// from tangent-space vector to world-space sample vector
	vec3 up        = abs(N.z) < 0.999 ? vec3(0.0, 0.0, 1.0) : vec3(1.0, 0.0, 0.0);
	vec3 tangent   = normalize(cross(up, N));
	vec3 bitangent = cross(N, tangent);
	
	vec3 sampleVec = tangent * H.x + bitangent * H.y + N * H.z; 
	return normalized(sampleVec);
}
```

This gives us a sample vector somewhat oriented around the expected microsurface's halfway vector based on some input roughness and the low-discrepancy sequence value `Xi`. Note that Epic Games uses the squared roughness for better visual results based on Disney's original PBR research. 

With the low-discrepancy Hammersley sequence and sample generation defined, we can finalize the pre-filter convolution shader.

```
#version 330 core
out vec4 FragColor; 
in vec3 localPos; 

uniform samplerCube environmentMap; 
uniform float roughness;

float RadicalInverse_VdC(uint bits);
vec2 Hammersley(uint i, uint N);
vec3 ImportanceSampleGGX(vec2 Xi, vec3 N, float roughness);

void main()
{
	vec3 N = normalize(localPos);
	vec3 R = N;
	vec3 V = R; 
	
	const uint SAMPLE_COUNT = 1024u;
	float totalWeight = 0.0;
	vec3 prefilteredColor = vec3(0.0);
	for(uint i = 0u; i < SAMPLE_COUNT; ++i)
	{
		vec2 Xi = Hammersley(i, SAMPLE_COUNT);
		vec3 H  = ImportanceSampleGGX(Xi, N, roughness);
		vec3 L  = normalize(2.0 * dot(V, H) * H - V);
		
		float NdotL = max(dot(N, L), 0.0);
		if(NdotL > 0.0)
		{
			prefilteredColor += texture(environmentMap, L).rgb * NdotL;
			totalWeight      += NdotL; 
		}
	}	
	prefilteredColor = prefilteredColor / totalWeight; 
	
	FragColor = vec4(prefilteredColor, 1.0);
}
```


We pre-filter the environment, based on some input roughness that varies over each mipmap level of the pre-filter cubemap (from $0.0$ to $1.0$), and store the result in `prefilteredColor`. The resulting `prefilteredColor` is divided by the total sample weight, where samples with less influence on the final result (for small `NdotL`) contribute less to the final weight. 

**Capturing Pre-Filter Mipmap Levels**

What's left to do is let OpenGL pre-filter the environment map with different roughness values over multiple mipmap levels. This is actually fairly easy to do with the original setup of the [irradiance](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) notes. 

```
prefilterShader.use();
prefilterShader.setInt("environmentMap", 0);
prefilterShader.setMat4("projection", captureProjection);
glActivateTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);

glBindFramebuffer(GL_FRAMEBUFFER, captureFBO);
unsigned int maxMipLevels = 5;
for (unsigned int mip = 0; mip < maxMipLevels; ++mip)
{
	// resize framebuffer according to mip-level size.
	unsigned int mipWidth  = 128 * std::pow(0.5, mip);
	unsigned int mipHeight = 128 * std::pow(0.5, mip);
	glBindRenderbuffer(GL_RENDERBUFFER, captureRBO);
	glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24, mipWidth,             mipHeight); 
	glViewport(0, 0, mipWidth, mipHeight);
	
	float roughness = (float)mip / (float)(maxMipLevels - 1);
	prefilterShader.setFloat("roughness", roughness);
	for (unsigned int i = 0; i < 6; ++i)
	{
		prefilterShader.setMat4("view", captureViews[i]);
		glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT_0,                      GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, prefilterMap, mip);
		glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
		renderCube();
	}
}
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```


This process is similar to the irradiance map convolution, but this time we scale the framebuffer's dimensions to the appropriate mipmap scale, each mip map reducing the dimensions by a scale of 2. Additionally, we specify the mip level we're rendering into in `glFramebufferTexture2D`'s last parameter and pass the roughness we're pre-filtering for to the pre-filter shader. 

This should give us a properly pre-filtered environment map that returns blurrier reflections the higher mip level we access it from. If we use the pre-filtered environment cubemap in the skybox shader and forcefully sample somewhat above its first mip level like so. 

`vec3 envColor = textureLod(environmentMap, WorldPos, 1.2).rgb;`

We get a result that indeed looks like a blurrier version of the original environment. 

![[Pasted image 20251217165803.png]]

If it looks somewhat similar you've successfully pre-filtered the HDR environment map. Play around with different mipmap levels to see the pre-filter map gradually change from sharp to blurry reflections on increasing mip levels. 

**Pre-filter Convolution Artifacts**

While the current pre-filter map works fine for most purposes, sooner or later you'll come across several render artifacts that are directly related to the pre-filter convolution. I'll list the most common here including how to fix them. 

**Cubemap Seams at High Roughness**

Sampling the pre-filter map on surfaces with a rough surface means sampling the pre-filter map on some of its lower mip levels. When sampling cubemaps, OpenGL doesn't linearly interpolate **across** cubemap faces. Because the lower mip levels are both of a lower resolution and the pre-filter map is convoluted with a much larger lobe, the lack of *between-cube-face* filtering becomes quite apparent.

![[Pasted image 20251218103517.png]]
 

Luckily for us, OpenGL gives us the option to properly filter across cubemap faces by enabling `GL_TEXTURE_CUBE_MAP_SEAMLESS`. 

`glEnable(GL_TEXTURE_CUBE_MAP_SEAMLESS);`

Simply enable this property somewhere at the start of your application and the seams will be gone. 

**Bright Dots in the Pre-filter Convolution**

Due to high frequency details and widely varying light intensities in specular reflections, convoluting the specular reflections requires a large numbers of samples to properly account for the widely varying nature of HDR environmental reflections. We already take a very large number of samples, but on some environments it may still not be enough at some of the rougher mip levels in which case you'll start seeing dotted patterns emerge around bright areas.

![[Pasted image 20251218104452.png]]

One option is to increase the sample count, but this won't be enough for all environments. As described by [Chetan Jags](https://chetanjags.wordpress.com/2015/08/26/image-based-lighting/) we can reduce this artifact by (during the pre-filter convolution) not directly sampling the environment map, but sampling a mip level of the environment map based on the integral's PDF and the roughness. 

```
float D = DistributionGGX(NdotH, roughness);
float pdf = (D * NdotH / (4.0 * HdotV)) + 0.0001;

float resolution = 512.0 // resolution of source cubemap (per face)
float saTexel    = 4.0 * PI / (6.0 * resolution * resolution);
float saSample   = 1.0 / (float(SAMPLE_COUNT) * pdf + 0.0001);

float mipLevel   = roughness == 0.0 ? 0.0 : 0.5 * log2(saSample / saTexel);
```

Don't forget to enable trilinear filtering on the environment map you want to sample its mip levels from. 

```
glBindTexutre(GL_TEXTURE_CUBE_MAP, envCubemap);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
```

And let OpenGL generate the mipmaps **after** the cubemap's base texture is set. 

```
// convert HDR equirectangular environment map to cubemap equivalent
[...]
// then generate mipmaps
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);
glGenerateMipmap(GL_TEXTURE_CUBE_MAP);
```

This works surprisingly well and should remove most, if not all, dots in your pre-filter map on rougher surfaces. 

**Pre-computing the BRDF**

With the pre-filtered environment up and running, we can focus on the second part of the split-sum approximation: the BRDF. Let's briefly review the specular split sum approximation again. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega}L_i(p, \omega_i)d \omega_i * \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d \omega_i}$

We've pre-computed the left part of the split sum approximation in the pre-filter map over different roughness levels. The right side requires us to convolute the BRDF equation over the angle $\Large{n \cdot \omega_o}$, the surface roughness, and Fresnel's $\Large{F_0}$. This is similar to integrating the specular BRDF with a solid-white environment or a constant radiance $\Large{L_i}$ of 1.0. Convoluting the BRDF over 3 variables is a bit much, but we can try to move $\Large{F_0}$ out of the specular BRDF equation. 

$\LARGE{\int\limits_{\Omega} f_r(p, \omega_i, \omega_o)n \cdot \omega_i d \omega_i = \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) \frac{F(\omega_o, h)}{F(\omega_o, h)} n \cdot \omega_i d \omega_i}$

With $\Large{F}$ being the Fresnel equation. Moving the Fresnel denominator to the BRDF gives us the following equivalent equation. 

$\LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} F(\omega_o, h) n \cdot \omega_i d \omega_i}$    
 
Substituting the right-most $\Large{F}$ with the Fresnel-Schlick approximation gives us.

$\LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 + (1 - F_0)(1 - \omega_o \cdot h)^5) n \cdot \omega_i d \omega_i}$  

Let's replace $\Large{(1 - \omega_o \cdot h)^5}$ by $\Large{a}$ to make it easier to solve for $\Large{F_0}$. 

$\LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 + (1 - F_0)a) n \cdot \omega_i d \omega_i}$

$\LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 + 1 * a - F_0 * a) n \cdot \omega_i d \omega_i}$

$\LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 * (1 - a) + a) n \cdot \omega_i d \omega_i}$

Then we split the Fresnel function $\Large{F}$ over two integrals. 

$\LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 * (1 - a)) n \cdot \omega_i d \omega_i} + \LARGE{\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (a) n \cdot \omega_i d \omega_i}$ 

This way, $\Large{F_0}$ is constant over the integral and we can take $\Large{F_0}$ out of the integral. Next, we substitute $\Large{a}$ back to its original form giving us the final split sum BRDF equation. 
$\LARGE{F_0 \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) (1 -(1 - \omega_o \cdot h)^5) n \cdot \omega_i d \omega_i} + \LARGE{\int\limits_{\Omega} f_r(p, \omega_i, \omega_o) (1 - \omega_o \cdot h)^5 n \cdot \omega_i d \omega_i}$
 The two resulting integrals represent a scale and a bias to $\Large{F_0}$ respectively. Note that as $\Large{f_r(p, \omega_i, \omega_o)}$ already contains a term for $\Large{F}$ they both cancel out, removing $\Large{F}$ from $\Large{f_r}$. 

In a similar fashion to the earlier convoluted environment maps, we can convolute the BRDF equations on their inputs: the angle between $\Large{n}$ and $\Large{\omega_o}$, and the roughness. We store the convoluted results in a 2D lookup texture (LUT) known as a **BRDF integration** map that we later use in our PBR lighting shader to get the final convoluted indirect specular result. 

The BRDF convolution shader operates on a 2D plane, using its 2D texture coordinates directly as inputs to the BRDF convolution (`NdotV` and `roughness`). The convolution code is largely similar to the pre-filter convolution, except that it now processes the sample vector according to our BRDF's geometry function and Fresnel-Schlick's approximation. 

```
vec2 IntegrateBRDF(float NdotV, float roughness)
{
	vec3 V;
	V.x = sqrt(1.0 - NdotV*NdotV);
	V.y = 0.0;
	V.z = NdotV;
	
	float A = 0.0;
	float B = 0.0;
	
	vec3 N = vec3(0.0, 0.0, 1.0);
	
	const uint SAMPLE_COUNT = 1024u;
	
	for(uint i = 0ul i < SAMPLE_COUNT; ++i)
	{
		vec2 Xi = Hammersley(i, SAMPLE_COUNT);
		vec3 H  = ImportanceSampleGGX(Xi, N, roughness);
		vec3 L  = normalize(2.0 * dot(V, H) * H - V);
		
		float NdotL = max(L.z, 0.0);
		float NdotH = max(H.z, 0.0);
		float VdotH = max(dot(V, H), 0.0);
		
		if(NdotL > 0.0)
		{
			float G = GeometrySmith(N, V, L, roughness);
			float G_Vis = (G * VdotH) / (NdotH * NdotV);
			float Fc = pow(1.0 - VdotH, 5.0);
			
			A += (1.0 - Fc) * G_Vis;
			B += Fc * G_Vis;
		}
	}
	A /= float(SAMPLE_COUNT);
	B /= float(SAMPLE_COUNT);
	return vec2(A, B);
}

void main()
{
	vec2 integrateBRDF = IntegrateBRDF(TexCoords.x, TexCoords.y);
	FragColor = integratedBRDF;
}
```

As you can see, the BRDF convolution is a direct translation from the mathematics to code. We take both the angle $\Large{\theta}$ and the roughness as input, generate a sample vector with importance sampling, process it over the geometry and the derived Fresnel term of the BRDF, and output both a scale and bias to $\Large{F_0}$ for each sample, averaging them in the end. 

You may recall from the [theory](https://learnopengl.com/PBR/Theory) notes that the geometry term of the BRDF is slightly different when used alongside IBL as its $\Large{k}$ variable has a slightly different interpolation. 

$\LARGE{k_{direct} = \frac{(a + 1)^2}{8}}$ 

 $\LARGE{k_{IBL} = \frac{a^2}{2}}$ 

Since the BRDF convolution is part of the specular IBL integral we'll use $\Large{k_{IBL}}$ for the Schlick-GGX geometry function.

```
float GeometrySchlickGGX(float NdotV, float roughness)
{
	float a = roughness 
}
```
 











