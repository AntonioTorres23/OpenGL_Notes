
In the [previous](https://learnopengl.com/PBR/Theory) notes we laid the foundation for getting a realistic physically based renderer off the ground. In these notes we'll focus on translating the previously discussed theory into an actual renderer that uses direct (or analytic) light sources: think of point lights, directional lights, and/or spotlights. 

Let's start by re-visiting the final reflectance equation from the previous notes. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$      

We now know mostly what's going on, but what still remained a big unknown is how exactly we're going to represent irradiance, the total radiance $\Large{L}$, of the scene. We know that radiance $\Large{L}$ (as interpreted in computer graphics land) measures the radiant flux $\Large{\phi}$ or light energy of a light source over a given solid angle $\Large{\omega}$. In our case we assumed the solid angle $\Large{\omega}$ to be infinitely small in which case radiance measures the flux of a light source over a single light ray or direction vector. 

Given this knowledge, how do we translate this into some of the lighting knowledge we've accumulated from previous notes? Well, imagine we have a single point (a light source that shines equally bright in all directions) with a radiant flux of $(23.47, )$

