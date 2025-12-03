
We've briefly touched the topic in the basic lighting notes: ambient lighting. Ambient lighting is a fixed light constant we add to the overall lighting of a scene to simulate the scattering of light. In reality, light scatters in all kinds of directions with varying intensities so the indirectly lit parts of a scene should also have varying intensities. One type of lighting approximation is called **ambient occlusion** that tries to approximate indirect lighting by darkening creases, holes, and surfaces that are close to each other. These areas are largely occluded by surrounding geometry and thus light rays have fewer places to escape to, hence the areas appear darker. Take a look at the corners and creases of your room to see that the light there seems just a little darker. 

Below is an example image of a scene with and without ambient occlusion. Notice how especially between the creases, the (ambient) light is more occluded. 

![[Pasted image 20251202143708.png]]

While not an incredibly obvious effect, the image with ambient occlusion enabled does feel a lot more realistic due to these small occlusion-like details, giving the entire scene a greater field of depth. 

Ambient occlusion techniques are expensive as they have to take surrounding geometry into account. One could shoot a large number of rays for each point in space to determine its amount of occlusion, but that quickly becomes computationally infeasible for real-time solutions. In 2007, Crytek published a technique called **Screen-Space Ambient Occlusion** (SSAO) for use in their title *Crysis*. The technique uses a depth buffer in screen-space to determine the amount of occlusion instead of real geometrical data. The approach is incredibly fast compared to real ambient occlusion and gives plausible results, making it the de-facto standard for approximating real-time ambient occlusion. 

The basics behind screen-space ambient occlusion are simple: for each fragment on a screen-filled quad we calculate an **occlusion factor** based on the fragment's surrounding depth values. The occlusion factor is then used to reduce or nullify the fragment's ambient lighting component. The occlusion factor is obtained by taking multiple depth samples in a sphere sample kernel surrounding the fragment position and compare each of the samples with the current fragment's depth value. The number of samples that have a higher depth value than the fragment's depth represents the occlusion factor. 

![[Pasted image 20251202152255.png]]

Each of the gray depth samples that are inside geometry contribute to the total occlusion factor; the more samples we find inside geometry, the less ambient lighting the fragment should eventually receive. 

It is clear the quality and precision of the effect directly relates to the number of samples we take. If the sample count is too low, the precision drastically reduces and we get an artifact called **banding**; if it is too high, we lose performance. We can reduce the amount of samples we have to test by introducing some randomness into the sample kernel. By randomly rotating the sample kernel each fragment we can get high quality results with a much smaller amount of samples. This does come at a price as the randomness introduces a noticeable **noise pattern** that we'll have to fix by blurring the results. Below is an image (courtesy of John Chapman) showcasing the banding effect and the effect randomness has on the results. 

![[Pasted image 20251202155548.png]]

As you can see, even though we get noticeable banding on the SSAO results due to the low sample count, by introducing some randomness the banding effects are completely gone. 

The SSAO method developed by Crytek had a certain visual style. Because the sample kernel used was a sphere, it caused flat walls to look gray as half of the kernel samples end up being in the surrounding geometry. Below is an image of Crysis's screen-space ambient occlusion that clearly portrays this gray feel. 

![[Pasted image 20251202160810.png]]

For that reason we won't be using a sphere sample kernel, but rather a hemisphere sample kernel oriented along a surface's normal  vector. 

![[Pasted image 20251202160907.png]]
By sampling around this **normal oriented hemisphere** we do not consider the fragment's underlying geometry to be a contribution to the occlusion factor. This removes the gray-feel of ambient occlusion and generally produces more realistic results. This note's technique is based on this normal-oriented hemisphere method and a slightly modified version of John Chapman's brilliant [SSAO tutorial](http://john-chapman-graphics.blogspot.nl/2013/01/ssao-tutorial.html).

**Sample Buffers**

SSAO requires geometrical info as we need some way to determine the occlusion factor of a fragment. For each fragment, we're going to need the following data. 

- A per-fragment **position** vector.
- A per-fragment **normal** vector.
- A per-fragment **albedo** color. 
- A **sample kernel**.
- A per-fragment **random rotation** vector used to rotate the sample kernel. 

Using a per-fragment view-space position we can orient a sample hemisphere kernel around the fragment's view-space surface normal and use this kernel to sample the position buffer texture at varying offsets. For each per-fragment kernel sample we compare its depth with its depth in the position buffer to determine the amount of occlusion. The resulting occlusion factor is then used to limit the final ambient lighting component. By also including a per-fragment rotation vector we can significantly reduce the number of samples we'll need to take as we'll soon see. 

![[Pasted image 20251202163858.png]]

As SSAO is a screen-space technique we can calculate its effect on each fragment on a screen-filled 2D quad. This does mean we have no geometrical information of the scene. What we could do, is render the geometrical per-fragment data into screen space-textures that we then later send to the SSAO shader so we have access to the per-fragment geometrical data. If you've followed along with the previous notes you'll realize this looks quite like a deferred render's G-buffer setup. For that reason SSAO is perfectly suited in combination with deferred rendering as we already have the position and normal vectors in the G-buffer. 

In these notes we're going to implement SSAO on top of a slightly simplified version of the deferred renderer from the deferred shading notes. If you're not sure what deferred shading is, be sure to first read up on that.

As we should have per-fragment position and normal data available from the scene objects, the fragment shader of the geometry stage is fairly simple. 

```
#version 330 core 
layout (location = 0) in vec3 gPosition;
layout (location = 1) in vec3 gNormal;
layout (location = 2) in vec4 gAlbedospec;

in vec2 TexCoords;
in vec3 FragPos;
in vec3 Normal

void main()
{
	// store the fragment position vector in the first gbuffer texture
	gPosition = FragPos; 
	// also store the per-fragment normals into the gbuffer
	gNormal = normalize(Normal);
	// an
	
}
```

