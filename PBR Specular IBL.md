
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


The second part of the split sum equation equals the BRDF part of the specular integral. If we pretend the incoming radiance is completely white for every direction (thus $\Large{L(p, x) = 1.0}$) we can pre-calculate the BRDF's response given an input roughness and an input angle between the normal $\Large{n}$ and the light direction $\Large{\omega_i}$, or $\Large{n \cdot \omega_i}$. Epic Games stores the pre-computed BRDF's response to each normal and light direction combination on varying roughness values in a 2D lookup texture (LUT) known as the **BRDF integration** map. The 2D lookup texture outputs a s




































