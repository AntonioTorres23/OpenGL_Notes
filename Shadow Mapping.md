
Shadows are a result of the absence of light due to occlusion. When a light source's rays do not hit an object because it gets occluded by some other object, the object is in shadow. Shadows add a great deal of realism to a lit scene and make it easier for a viewer to observe spatial relationships between objects. They give a greater sense of depth to your scene and objects. For example, take a look at the following image of a scene with and without shadows. 

![[Pasted image 20251021094528.png]]

You can see that with shadows it becomes much more obvious how objects relate to each other. For instance, the fact that one of the cubes is floating above the others is only really noticeable when we have shadows. 

Shadows are a bit tricky to implement though, specifically because in current real-time (rasterized graphics) research a perfect shadow algorithm hasn't been developed yet. There are several good shadow approximation techniques, but they all have their little quirks and annoyances which we have to take into account. 

One technique used by most videogames that gives decent results and is relatively easy to implement is **shadow mapping**. Shadow mapping is not too difficult to understand, doesn't cost too much in performance and quite easily extends into more advanced algorithms (like [Omnidirectional Shadow Maps](https://learnopengl.com/Advanced-Lighting/Shadows/Point-Shadows) and [Cascaded Shadow Maps](https://learnopengl.com/Guest-Articles/2021/CSM)). 

**Shadow Mapping**