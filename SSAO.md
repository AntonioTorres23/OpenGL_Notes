
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