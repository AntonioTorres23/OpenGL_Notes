
In the [previous](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) notes we've set up PBR in combination with image based lighting by pre-computing an irradiance map as the lighting's indirect diffuse portion. In these notes we'll focus on the specular part of the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + k_s \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$

You'll notice that the Cook-Torrance specular portion (multiplied by $\Large{kS}$) isn't constant over the integral and is dependent on the incoming light direction, but **also** the incoming view direction. Trying to solve the integral for all incoming light including all possible view directions is a combinatorial overload and way too expensive to calculate on a real-time basis. Epic Games proposed a solution where they were able to pre-convolute the specular part for real-time purposes, given a few compromises, known as the **split sum approximation**. 

The split sum approximation splits the specular part of the reflectance equation into two separate parts that we can individually convolute and later