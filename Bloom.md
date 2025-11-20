
Bright light sources and brightly lit regions are often difficult to convey to the viewer as the intensity range of a monitor is limited. One way to distinguish bright light sources on a monitor is by making them glow; the light then *bleeds* around the light source. This effectively gives the viewer the illusion these light sources or bright regions are intensely bright. 

This light bleeding, or glow effect, is achieved with a pos-processing effect called **Bloom**. Bloom gives all brightly lit regions of a scene a glow-like effect. An example of a scene with and without glow can be seen below. 

![[Pasted image 20251120151725.png]]

Bloom gives noticeable visual cues about the brightness of objects. When done in a subtle fashion (which some games drastically fail to do) Bloom significantly boosts the lighting of your scene and allows for a large range of dramatic effects. 

Bloom works best in combination with HDR rendering. A common misconception is that HDR is the same as Bloom as many people use the terms interchangeably. There are however completely different techniques used for different purposes. It is possible to implement Bloom with default 8-bit precision framebuffers, just as it is possible to use HDR without the Bloom effect. It is simply that HDR makes Bloom more effective to implement (as we'll later see) 

To implement Bloom, we render a lit scene as usual and extract both the scene's HDR color buffer and an image of the scene with only its bright regions visible. This extracted brightness image is then blurred and the result added on top of the original HDR scene image. 

Let's illustrate this process in a step by step fashion. We render a scene filled with 4 bright light sources, visualized as colored cubes. The colored light cubes have a brightness values between $1.5$ and $15.0$. If we were to render this to an HDR color buffer the scene looks as follows. 

![[Pasted image 20251120152620.png]]

We take this HDR color buffer texture and extract all the fragments that exceed a certain brightness. This gives us an image that only show the bright colored regions as their fragment intensities exceed a certain threshold. 

![[Pasted image 20251120152800.png]]

We take this thresholded brightness texture and blur the result. The strength of the bloom effect is largely determined by the range and strength of the blur filter used. 

![[Pasted image 20251120152900.png]]

The resulting blurred texture is what we use to get the glow or light-bleeding effect. This blurred texture is added on top of the original HDR scene texture. Because the bright regions are extended in both width and height due to the blur filter, the bright regions of the scene appear to glow or *bleed* light. 

![[Pasted image 20251120153057.png]]









































