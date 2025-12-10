
PBR, or more commonly known as **physically based rendering**, is a collection of render techniques that are more or less based on the same underlying theory that more closely matches that of the physical world. As physically based rendering aims to mimic light in a physically plausible way, it generally looks more realistic compared to our original lighting algorithms like Phong and Blinn-Phong. Not only does it look better, as it closely approximates actual physics, we (and especially the artists) can author surface materials based on physical parameters without having to resort to cheap hacks and tweaks to make the lighting look right. One of the bigger advantages of authoring materials based on physical parameters is that these materials will look correct regardless of lighting conditions; something that is not true in non-PBR pipelines. 

Physically based rendering is still nonetheless an approximation of reality (based on the principles of physics) which is why it's not called physical shading, but physically *based* shading. For a PBR lighting model to be considered physically based, it has to satisfy the following 3 conditions (don't worry we'll get to them soon enough).

1. Be based on the microfacet surface model. 
2. Be energy conserving.
3. Use a physically based BRDF

In the next PBR notes we'll be focusing on the PBR approach as originally explored by Disney and adopted for real-time display by Epic Games. Their approach, based on the **metallic workflow**, is decently documented, widely adopted on most popular engines, and look visually amazing. By the end of these notes we'll have something that looks like this. 

![[Pasted image 20251208163052.png]]

Keep in mind, the topics in these chapters are rather advanced so it is advised to have a good understanding of OpenGL and shader lighting. Some of the more advanced knowledge you'll need for this series are: [framebuffers](https://learnopengl.com/Advanced-OpenGL/Framebuffers), [cubemaps](https://learnopengl.com/Advanced-OpenGL/Cubemaps), [gamma correction](https://learnopengl.com/Advanced-Lighting/Gamma-Correction), [HDR](https://learnopengl.com/Advanced-Lighting/HDR), and [normal mapping](https://learnopengl.com/Advanced-Lighting/Normal-Mapping). We'll also delve into some advanced mathematics, but I'll do my best to explain the concepts as clear as possible. 

**The Microfacet Model**

All PBR techniques are based on the theory of microfacets. The theory describes that any surface at a microscopic scale can be described by tiny little perfectly reflective mirrors called **microfacets**. Depending on the roughness of a surface, the alignment of these tiny little mirrors can differ quite a lot. 

![[Pasted image 20251208163554.png]]

The rougher a surface is, the more chaotically aligned each microfacet will be along the surface. The effect of these tiny-like mirror alignments is, that when specifically talking about specular lighting/reflection, the incoming light rays are more likely to **scatter** along completely different directions on rougher surfaces, resulting in a more widespread specular reflection. In contrast, on a smooth surface the light rays are more likely to reflect in roughly the same direction, giving us smaller and sharper reflections. 

![[Pasted image 20251208164359.png]]

No surface is completely smooth on a microscopic level, but seeing as these microfacets are small enough that we can't make a distinction between them on a per-pixel basis, we statistically approximate the surface's microfacet roughness given a **roughness** parameter. Based on the roughness of a surface, we can calculate the ratio of microfacets roughly aligned to some vector $h$. This vector $h$ is the **halfway vector** that sits halfway between the light $l$ and view $v$ vector. We've discussed the halfway vector before in the [advanced lighting](https://learnopengl.com/Advanced-Lighting/Advanced-Lighting) notes which is calculated as the sum of $l$ and $v$ divided by its length (normalized).

$\LARGE{h = \frac{l + v}{\|l + v\|}}$ 

The more the microfacets are aligned to the halfway vector, the sharper and stronger the specular reflection. Together with a roughness parameter that varies between 0 and 1, we can statistically approximate the alignment of the microfacets. 

![[Pasted image 20251208170446.png]]

We can see that higher roughness values display a much larger specular reflection shape, in contrast with the smaller and sharper reflection shape on smooth surfaces. 

**Energy Conservation**

The microfacet approximation employs a form of **energy conservation**: outgoing light energy should never exceed the incoming light energy (excluding emissive surfaces). Looking at the above image we see the specular reflection area increase, but also its brightness decrease at increasing roughness levels.  If the specular intensity were to be the same at each pixel (regardless of size of the specular shape) the rougher surfaces would emit much more energy, violating the energy conservation principle. This is why we see specular reflections more intensely on smooth surfaces and more dimly on rough surfaces. 

For energy conservation to hold, we need to make a clear distinction between diffuse and specular light. The moment a light ray hits a surface, it gets split in both a **refraction** part and a **reflection** part. The reflection part is light that directly gets reflected and doesn't enter the surface; this is what we know as specular lighting. The refraction part is the remaining light that enters the surface and gets absorbed; this is what we know as diffuse lighting. 

There are some nuances here as refracted light doesn't immediately get absorbed by touching the surface. From physics, we know that light can be modeled as a beam of energy that keeps moving forward until it loses all of its energy; the way a light beam loses energy is by collision. Each material consists of tiny little particles that can collide with the light ray as illustrated in the following image. The particles absorb some, or all, of the light's energy at each collision which is converted into heat. 

![[Pasted image 20251209112017.png]]

Generally, not all energy is absorbed and the light will continue to **scatter** in a (mostly) random direction at which point it collides with other particles until its energy is depleted or it leaves the surface again. Light rays re-emerging out of the surface contribute to the surface's observed (diffuse) color. In physically based rendering however, we make the simplifying assumption that all refracted light gets absorbed and scattered at a very small area of impact, ignoring the effect of scattered light rays that would've exited the surface at a distance. Specific shader techniques that do take this into account are known as **subsurface scattering** techniques that significantly improve the visual quality on materials like skin, marble, or way, but come at the price of performance. 

An additional subtlety when it comes to reflection and refraction are surfaces that are **metallic**. Metallic surfaces react different to light compared to non-metallic surfaces (also known as dielectrics). Metallic surfaces follow the same principles of reflection and refraction, but **all** refracted light gets directly absorbed without scattering. This means metallic surfaces only leave reflected or specular light; metallic surfaces show no diffuse colors. Because of this apparent distinction between metals and dielectrics, they're both treated differently in the PBR pipeline which we'll delve into further down the notes. 

This distinction between reflected and refracted lights brings us to another observation regarding energy preservation: they're **mutually exclusive**. Whatever light energy gets reflected will no longer be absorbed by the material itself. Thus, the energy left to enter the surface as refracted light is directly the resulting energy after we've taken reflection into account. 

We preserve this energy conserving relation by first calculating the specular fraction that amounts the percentage the incoming light's energy is reflected. The fraction of refracted light is then directly calculated from the specular fraction as.

```
float kS = calculateSpecularComponent(...); // reflection/specular fraction
float kD = 1.0 - kS;                        // refraction/diffuse fraction
```

This way we know both the amount the incoming light reflects and the amount the incoming light refracts, while adhering to the energy conservation principle. Given this approach, it is impossible for both the refracted/diffuse and reflected/specular contribution to exceed $1.0$, thus ensuring the sum of their energy never exceeds the incoming light energy. Something we did not take into account in previous lighting notes. 

**The Reflectance Equation**

This brings us to something called the [render equation](https://en.wikipedia.org/wiki/Rendering_equation), an elaborate equation some very smart folks out there came up with that is currently the best model we have for simulating the visuals of light. Physically based rendering strongly follows a more specialized version of the render equation known as the **reflectance equation**. To properly understand PBR, it's important to first build a solid understanding of the reflectance equation. 


$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p, \omega_i) n \cdot \omega_i d\omega_i}$

The reflectance equation appears daunting at first, but as we'll dissect it you'll see it slowly starts to make sense. To understand the equation, we have to delve into a bit of **radiometry**. Radiometry is the measurement of electromagnetic radiation, including visible light. There are several radiometric quantities we can use to measure light over surfaces and directions, but we will only discuss a single one that's relevant to the reflectance equation known as **radiance**, denoted here as $L$. Radiance is used to quantify the magnitude or strength of light coming form a single direction. It's a bit tricky to understand at first as radiance is a combination of multiple physical quantities so we'll focus on those first.

**Radiant Flux**: radiant flux $\Large{\Phi}$ is the transmitted energy of a light source measured in Watts. Light is a collective sum of energy over multiple different wavelengths, each wavelength associated with a particular (visible) color. The emitted energy of a light source can therefore be thought of as a function of all its different wavelengths. Wavelengths between 390nm to 700nm (nanometers) are considered to be part of the visible light spectrum i.e. wavelengths the human eye is able to perceive. Below you'll find an image of the different energies per wavelength of daylight. 


![[Pasted image 20251209132934.png]]

The radiant flux measures the total area of this function of different wavelengths. Directly taking this measure of wavelengths as input is slightly impractical so we often make the simplification of representing radiant flux, not as a function of varying wavelength strengths, but as a light color triplet encoded as RGB (or as we'd commonly call it: light color). This encoding does come at quite a loss of information, but this is generally negligible for visual aspects. 

**Solid Angle:** the solid angle, denoted as $\Large{\omega}$, tells us the size or area of a shape projected onto a unit sphere. The area of the projected shape onto this unit sphere is known as the **solid angle**; you can visualize the solid angle as a direction with volume. 

![[Pasted image 20251209135544.png]]

Think of being an observer at the center of this unit sphere and looking in the direction of the shape; the size of the silhouette you make out of it is the solid angle. 

**Radiant Intensity**: radiant intensity measures the amount of radiant flux per solid angle, or the strength of a light source over a projected area onto the unit sphere. For instance, given an omnidirectional light that radiates equally in all directions, the radiant intensity gives us its energy over a specific area (solid angle). 

![[Pasted image 20251209141952.png]]

The equation to describe the radiant intensity is defined as follows.

$\LARGE{I = \frac{d\Phi}{d\omega}}$

Where $I$ is the radiant flux $\Large{\Phi}$ over the solid angle $\Large{\omega}$.

With knowledge of radiant flux, radiant intensity, and the solid angle, we can finally describe the equation for **radiance**. Radiance is described as the total observed energy in an area $A$ over the solid angle $\Large{\omega}$ of a light of radiant intensity $\Large{\Phi}$. 

$\LARGE{L = \frac{d^2 \Phi}{d A d \omega \cos \theta}}$   

![[Pasted image 20251209144456.png]]

Radiance is a radiometric measure of the amount of light in an area, scaled by the **incident** (or incoming) angle $\large{\theta}$ of the light to the surface's normal as $\Large{\cos \theta}$: light is weaker the less it directly radiates onto the surface, and strongest when it is directly perpendicular to a surface. This is similar to our perception of diffuse lighting from the [basic lighting](https://learnopengl.com/Lighting/Basic-lighting) notes as $\Large{\cos \theta}$ directly corresponds to the dot product between the light's direction vector and the surface normal. 

`float cosTheta = dot(lightDir, N);`

The radiance equation is quite useful as it contains most physical quantities we're interested in. If we consider the solid angle $\Large{\omega}$ and the area $A$ to be infinitely small, we can use radiance to measure the flux of a single ray of light hitting a single point in space. This relation allow us to calculate the radiance of a single ray influencing a single (fragment) point; we effectively translate the solid angle $\Large{\omega}$ into a direction vector $\Large{\omega}$, and $A$ into a point $\Large{p}$. This way, we can directly use radiance in our shaders to calculate a single light ray's per-fragment contribution. 

In fact, when it comes to radiance we generally care about **all** incoming light onto point $\Large{p}$, which is the sum of all radiance known as **irradiance**. With knowledge of both radiance and irradiance we can get back to the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p, \omega_i) n \cdot \omega_i d\omega_i}$

We now know that $L$ in the render equation represents the radiance of some point $\Large{p}$ and some incoming infinitely small solid angle $\Large{\omega_i}$ which can be thought as an incoming direction vector $\Large{\omega_i}$. Remember that $\Large{\cos \theta}$ scales the energy based on the light's incident angle to the surface, which we find in the reflectance equation as $\Large{n \cdot \omega_i}$. The reflectance equation calculates the sum of reflected radiance $\Large{L_o(p, \omega_o)}$ of a point $\Large{p}$ in direction $\Large{\omega_o}$ which is the outgoing direction to the viewer. Or to put it differently: $\Large{L_o}$ measures the reflected sum of the lights' irradiance onto point $\Large{p}$ as viewed from $\Large{\omega_o}$. 

The reflectance equation is based around irradiance, which is the sum of all incoming radiance we measure light of. Not just of a single incoming light direction, but of all incoming light directions within a hemisphere $\Large{\Omega}$ centered around point $\Large{p}$. A **hemisphere** can be described as half a sphere aligned around a surface's normal $\Large{n}$. 

![[Pasted image 20251210100432.png]]

To calculate the total of values inside an area or (in this case a hemisphere) a volume, we use a mathematical construct called an **integral** denoted in the reflectance as $\Large{\int}$ over all incoming directions $\Large{d \omega_i}$, within the hemisphere $\Large{\Omega}$. An integral measures the area of a function, which can either be calculated analytically or numerically. As there is no analytical solution to both the render and reflectance equation, we'll want to numerically solve the integral discretely. This translates to taking the results of small discrete steps of the reflectance equation over the hemisphere $\Large{\Omega}$ and averaging their results over the step size. This is known as the **Riemann Sum** that we can roughly visualize in code as follows. 

```
int steps = 100;
float sum- 0.0f;
vec3 P  = ...;
vec3 Wo = ...;
vec3 N  = ...; 
float dW = 1.0f / steps; 

for(int i = 0; i < steps; ++i)
{
	vec3 Wi = getNextIncomingLightDir(i);
	sum += Fr(P, Wi, Wo) * L(P, Wi) * dot(N, Wi) * dW;
}
```

By scaling the steps by `dW`, the sum will equal the total area or volume of the integral function. The `dW` to scale each discreate step can be though of as $\Large{d\omega_i}$ in the reflectance equation. Mathematically $\Large{d\omega_i}$ is the continuous symbol over which we calculate the integral, and while it does not directly relate to `dW` in code (as this is a discrete step of the Riemann sum), it helps to think of it this way. Keep in mind that taking discrete steps will always give us an approximation of the total area of the function. A careful reading will notice we can increase the *accuracy* of the Riemann Sum by increasing the number of steps. 

The reflectance equation sums up the radiance of all incoming light directions $\Large{\omega_i}$ over the hemisphere $\Large{\Omega}$ scaled by $\Large{f_r}$ that hit point $\Large{p}$ and returns the sum of reflected light $\Large{L_o}$ in the viewer's direction. The incoming radiance can some from [light sources](https://learnopengl.com/PBR/Lighting) as we're familiar with, or from an environmental map measuring radiance of every incoming direction as we'll discuss in the [IBL](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) notes. 

Now the only unknown left is the $\Large{f_r}$ symbol known as **BRDF** or **bidirectional reflective distribution function** that scales or weights the incoming radiance based on the surface's material properties. 

**BRDF**

The **BRDF**, or **bidirectional reflective distribution function**, is a function that takes as input the incoming (light)
  