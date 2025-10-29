
In the shadow mapping notes we learned to create dynamic shadows with shadow mapping. It works great, but it's mostly suited for directional (or spot) lights as the shadows are generated only in the direction of the light source. It is therefore also known as **directional shadow mapping** as the depth (or shadow) map is generated from only the direction the light is looking at. 

What this section of notes will focus on is the generation of dynamic shadows in all surrounding directions. The technique we're using is perfect for point lights as a real point light would cast shadows in all directions. This technique is known as point (light) shadows or more formerly as **omnidirectional shadow maps**. 

This section of notes builds upon the previous shadow mapping notes, so unless you're familiar with traditional shadow mapping it is advised to read the shadow mapping section first. 

This technique is mostly similar to directional shadow mapping: we generate a depth map from the light's perspective(s), sample the depth map based on the current fragment position, and compare each fragment with the stored depth value to see whether it is in shadow. The main difference between directional shadow mapping and omnidirectional shadow 