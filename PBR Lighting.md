
In the [previous](https://learnopengl.com/PBR/Theory) notes we laid the foundation for getting a realistic physically based renderer off the ground. In these notes we'll focus on translating the previously discussed theory into an actual renderer that uses direct (or analytic) light sources: think of point lights, directional lights, and/or spotlights. 

Let's start by re-visiting the final reflectance equation from the previous notes. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$      

We now know mostly what's going on, but what still remained a big unknown is how exactly we're going to represent irradiance, the total radiance $\Large{L}$, of the scene. We know that radiance $\Large{L}$ (as interpreted in computer graphics land) measures the radiant flux $\Large{\phi}$ or light energy of a light source over a given solid angle $\Large{\omega}$. In our case we assumed the solid angle $\Large{\omega}$ to be infinitely small in which case radiance measures the flux of a light source over a single light ray or direction vector. 

Given this knowledge, how do we translate this into some of the lighting knowledge we've accumulated from previous notes? Well, imagine we have a single point light (a light source that shines equally bright in all directions) with a radiant flux of $(23.47, 21.31, 20.79)$ as translated to an RGB triplet. The radiant intensity of this light source equals its radiant flux at all outgoing direction rays. However, when shading a specific point $\Large{p}$ on a surface, of all possible incoming light directions over its hemisphere $\Large{\Omega}$, only one incoming direction vector $\Large{\omega_i}$ directly comes from the point light source. As we only have a single light source in our scene, assumed to be a single point in space, all other possible incoming light directions have zero radiance observed over the surface point $\Large{p}$. 

![[Pasted image 20251211164143.png]]
If at first, we assume that light attenuation (dimming of light over distance) does not affect the point light source, the radiance of the incoming light ray is the same regardless of where we position the light (excluding scaling radiance by the incident angle $\Large{\cos\theta}$). This, because the point light has the same radiant intensity regardless of the angle we look at, effectively modeling its radiant intensity as its radiant flux: a constant vector $(23.47, 21.31, 20.79)$. In layman's terms (I believe), the radiant intensity (how strong the light is) is the same as it's radiant flux (the color of the light). Meaning there is no drop in color/light quality (I think). 

However, radiance also takes a position $\Large{p}$ as input and as any realistic point light source takes light attenuation into account, the radiant intensity of the point light source is scaled by some measure of the distance between point $\Large{p}$ and the light source. Then, as extracted from the original radiance equation, the result is scaled by the dot product between the surface normal $\Large{n}$ and the incoming light direction $\Large{\omega_i}$. 

To put this in more practical terms: in the case of a direct point light the radiance function $\Large{L}$ measures the light color, attenuated over its distance to $\Large{p}$ and scaled by $\Large{n \cdot \omega_i}$, but only over the single light ray $\Large{\omega_i}$ that hits $\Large{p}$ which equals the light's direction vector from $\Large{p}$. In code this translates to. 

```
vec3 lightColor = vec3(23.47, 21.31, 20.79);
vec3 wi         = normalize(lightPos - FragPos);
float cosTheta  = max(dot(N, Wi), 0.0);
float attenuation = calculateAttenuation(FragPos, lightPos);
vec3 radiance   = lightColor * attenuation * cosTheta;  
```

Aside from the different terminology, this piece of code should look awfully familiar to you: this is exactly how we've been doing diffuse lighting so far. When it comes to direct lighting, radiance is calculated similarly to how we've calculated lighting before as only a single light direction vector contributes to the surface's radiance. 

Note that this assumption holds as point lights are infinitely small and only a single point in space. If we were to model a light that has area or volume, its radiance would be non-zero in more than one incoming light direction. 

For other types of light sources originating from a single point we calcula
