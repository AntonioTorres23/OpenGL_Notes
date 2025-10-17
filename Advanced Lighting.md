
In the lighting notes we briefly introduced the Phong lighting model to bring a basic amount of realism into our scenes. The Phong model looks nice, but has a few nuances we'll focus on in this note section.

**Blinn-Phong**

Phong lighting is a great and very efficient approximation of lighting, but its specular reflection breakdown in certain conditions, specifically when the shininess property is low resulting in a large (rough) specular area. 
The image below shows what happens when we use a specular shininess exponent of 1.0 on a flat textured plane. 

![[Pasted image 20251017102402.png]]

YOou 