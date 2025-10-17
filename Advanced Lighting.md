
In the lighting notes we briefly introduced the Phong lighting model to bring a basic amount of realism into our scenes. The Phong model looks nice, but has a few nuances we'll focus on in this note section.

**Blinn-Phong**

Phong lighting is a great and very efficient approximation of lighting, but its specular reflection breakdown in certain conditions, specifically when the shininess property is low resulting in a large (rough) specular area. 
The image below shows what happens when we use a specular shininess exponent of 1.0 on a flat textured plane. 

![[Pasted image 20251017102402.png]]

You can see at the edges that the specular area is immediately cut off. The reason this happens is because the angle between the view and reflection vector doesn't go over 90 degrees. If the angle is larger than 90 degrees, the resulting dot product becomes negative and this results in a specular exponent of 0.0. You're probably thinking this won't be a problem since we shouldn't get any light with angles higher than 90 degrees anyways, right?

Wrong, this only applies to the diffuse component where an angle higher than 90 degrees between the normal and light source means the light source is below the lighted surface and thus the light's diffuse contribution should equal 0.0. However, with specular lighting we're not measuring the angle between light source and the normal, but between the view and refection vector. Take a look at the following two images. 

![[Pasted image 20251017132222.png]]

Here the issue should become apparent. The left image shows Phong reflections as familiar. With the theta 