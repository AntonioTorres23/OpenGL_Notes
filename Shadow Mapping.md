
Shadows are a result of the absence of light due to occlusion. When a light source's rays do not hit an object because it gets occluded by some other object, the object is in shadow. Shadows add a great deal of realism to a lit scene and make it easier for a viewer to observe spatial relationships between objects. They give a greater sense of depth to your scene and objects. For example, take a look at the following image of a scene with and without shadows. 

![[Pasted image 20251021094528.png]]

You can see that with shadows it becomes much more obvious how objects relate to each other. For instance, the fact that one of the cubes is floating above the others is only really noticeable when we have shadows. 

Shadows are a bit tricky to implement though, specifically because in current real-time (rasterized graphics) research a perfect shadow algorithm hasn't been developed yet. There are several good shadow approximation techniques, but they all have their little quirks and annoyances which we have to take into account. 

One technique used by most videogames that gives decent results and is relatively easy to implement is **shadow mapping**. Shadow mapping is not too difficult to understand, doesn't cost too much in performance and quite easily extends into more advanced algorithms (like [Omnidirectional Shadow Maps](https://learnopengl.com/Advanced-Lighting/Shadows/Point-Shadows) and [Cascaded Shadow Maps](https://learnopengl.com/Guest-Articles/2021/CSM)). 

**Shadow Mapping**

The idea behind shadow mapping is quite simple: we render the scene from the light's point of view and everything we see from the light's perspective is lit and everything we can't see must be in shadow. Imagine a floor section with a large box between itself and a light source. Since the light source will see this box and not the floor section when looking in its direction that specific floor section would be in shadow.

![[Pasted image 20251021100349.png]]

Here all the blue lines represent the fragments that the light source can see. The occluded fragments are shown as black lines: these are rendered as being shadowed. If we were to draw a line or **ray** from the light source to a fragment on the right-most box we can see the ray first hits the floating container before hitting the right-most container. As a result, the floating container's fragments is not lit and thus in shadow. 

We want to get the point on the ray where it first hit an object and compare this *closest point* to other points on this ray. We then do a basic test to see if a test point's ray position is further down the ray than the closest point and if so, the test point must be in shadow. Iterating through possibly thousands of light rays from such a light source is an extremely inefficient approach and doesn't lend itself too well for real-time rendering. We can do something similar, but without casting light rays. Instead, we can use something we're quite familiar with: the depth buffer. 

You may remember from the depth testing notes that a value in the depth buffer corresponds to the depth of a fragment clamped to $[0,1]$ from the camera's point of view. What if we were to render the scene from the light's perspective and store the resulting depth values in a texture? This way, we can sample the closest depth values as seen from the light's perspective. After all, the depth values show the first fragment visible from the light's perspective. We store all these depth values in a texture that we call a **depth map** or **shadow map**. 

![[Pasted image 20251021101852.png]]

The left image shows a directional light source (all light rays are parallel) casting a shadow on the surface below the cube. Using the depth values stored in the depth map we find the closest point and use that to determine whether fragments are in shadow. We create the depth map by rendering the scene (from the light's perspective) using a **view** and **projection** matrix specific to that light source. This projection and view matrix together form a transformation $T$ that transforms any 3D position to the light's (visible) coordinate space. 

A directional light doesn't have a position as it's modelled to be infinitely far away. However, for the sake of shadow mapping we need to render the scene from the light's perspective and thus render the scene from a position somewhere along the lines of the light direction. 