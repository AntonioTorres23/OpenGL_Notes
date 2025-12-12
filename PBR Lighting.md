
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

For other types of light sources originating from a single point we calculate radiance similarly. For instance, a directional light source has a constant $\Large{\omega_i}$ without an attenuation factor. And a spotlight would not have a constant radiant intensity, but one that is scaled by the forward direction vector of the spotlight. 

This also brings us back to the integral $\Large{\int}$ over the surface's hemisphere $\Large{\Omega}$. As we know beforehand the single locations of all contributing light sources while shading a single surface point, it is not required to try and solve the integral. We can directly take the (known) number of light sources and calculate their total irradiance, given that each light source has only a single light direction that influences the surface's radiance. This makes PBR on direct light sources relatively simple as we effectively only have to loop over the contributing light sources. When we later take environment lighting into account in the [IBL](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) we do have to take the integral into account as light can come in any direction. 

**A PBR Surface Model**

Let's start by writing a fragment shader that implements the previously described PBR models. First, we need to take the relevant PBR inputs required for shading the surface. 

```
#version 330 core
out vec4 FragColor; 
in vec2 TexCoords;
in vec3 WorldPos;
in vec3 Normal; 

uniform vec3 camPos; 

uniform vec3 albedo; 
uniform float metallic; 
uniform float roughness; 
uniform float ao; 
```

We take the standard inputs as calculated from a generic vertex shader and a set of constant material properties over the surface of the object. 

Then at the start of the fragment shader we do the usual calculations required for any lighting algorithm. 

```
void main()
{
	vec3 N = normalize(Normal);
	vec3 V = normalize(camPos - WorldPos);
	[...]
}
```

**Direct Lighting**

In this note's example demo we have 4 point lights that together represent the scene's irradiance. To satisfy the reflectance equation we loop over each light source, calculate its individual radiance and sum its contribution scaled by the BRDF and the light's incident angle. We can think of the loop as solving the integral $\Large{\int}$ over $\Large{\Omega}$ for direct light sources. First, we calculate the relevant per-light variables. 

```
vec3 Lo = vec3(0.0);
for(int i = 0; i < 4; ++i)
{
	vec3 L = normalize(lightPositions[i] - WorldPos); // WorldPos = FragPos? 
	vec3 H = normalize(V + L); // H = Halfway Direction; V = ViewPos; L = LightDir
	
	float distance    = length(lightPositions[i] - WorldPos); 
	float attenuation = 1.0 / (distance * distance);
	vec3 radiance     = lightColors[i] * attenuation;
	[...]
}
```

As we calculate lighting in linear space (we'll [gamma correct](https://learnopengl.com/Advanced-Lighting/Gamma-Correction) at the end of the shader) we attenuate the light sources by the more physically correct **inverse-square law**. 

While physically correct, you may still want to use the constant-linear-quadratic attenuation equation that (while not physically correct) can offer you significantly more control over the light's energy fall off. 

Then, for each light we want to calculate the full Cook-Torrance specular BRDF term.

$\LARGE{\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}}$

The first thing we want to do is calculate the ratio between the specular and diffuse reflection, or how much the surface reflects versus how much it refracts light. We know from the previous notes that the Fresnel equation calculates just that (note the clamp here to prevent black spots). 

```
vec3 fresnelSchlick(flat cosTheta, vec3 F0)
{
	return F0 + (1.0 - F0) * pow(clamp(1 - cosTheta, 0.0, 1.0,) 5.0);
}
```

The Fresnel-Schlick approximation expects a `F0` parameter which is known as the *surface reflection* at zero incidence or how much the surface reflects if looking di


