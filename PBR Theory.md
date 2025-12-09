
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

The microfacet approximation employs a form of **energy conservation**: outgoing light energy should never exceed the incoming light energy (excluding emissive surfaces). Looking at the above image we see the specular reflection area increase, but also its brightness decrease at increasing roughness levels.  If the specular intensity were to be the same at each pixel (regardless of size of the specular shape) the rougher surfaces would emit much more 




