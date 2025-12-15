
IBL, or **image based lighting**, is a collection of techniques to light objects, not by direct analytical lights as in the [previous](https://learnopengl.com/PBR/Lighting) notes, but by treating the surrounding environment as one big light source. This is generally accomplished by manipulating a cubemap environment map (taken from the real world or generated from a 3D scene) such that we can directly use it in our lighting equations: treating each cubemap texel as a light emitter. This way we can effectively capture an environment's global lighting and general feel, giving objects a better sense of *belonging* in their environment. 

As image based lighting algorithms capture the lighting of some (global) environment, its input is considered a more precise form of ambient lighting, even a crude approximation of global illumination. This makes IBL interesting for PBR as objects look significantly more physically accurate when we take the environments lighting into account. 

To start introducing IBL into our system let's take a quick look at the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$    

As described before, our main goal is to solve the integral of all incoming light directions $\Large{\omega_i}$ over the hemisphere $\Large{\Omega}$. Solving the integral in the previous notes was easy as we knew beforehand the exact few light directions $\Large{\omega_i}$ that contributed to the integral. This time however, **every** incoming light direction $\Large{\omega_i}$ from the surrounding environment could potentially have some radiance making it less trivial to solve the integral. This gives us two main requirements for solving the integral. 

- We need some way to retrieve the scene's radiance given any direction vector $\Large{\omega_i}$. 
- Solving the integral needs to be fast and real-time

Now, the first requirement is relatively easy. We've already hinted it, but one way of representing an environment or scene's irradiance is in the form of a (processed) environment cubemap. Given such a cubemap, we can visualize every texel of the cubemap as one single emitting light source. By sampling this cubemap with any direction vector $\Large{\omega_i}$, we retrieve the scene's radiance from that direction. 

Getting the scene's radiance given any direction vector $\Large{\omega_i}$ is then as simple as. 

`vec3 radiance = texture(_cubemapEnvironment, w_i).rgb;`

Still, solving the integral requires us to sample the environment map from not just one direction, but all possible directions $\Large{\omega_i}$ over the hemisphere $\Large{\Omega}$ which is far too expensive for each fragment shader invocation. To solve the integral in a more efficient fashion we'll want to *pre-process* or **pre-compute** most of the computations. For this we'll have to delve a bit deeper into the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$ 

Taking a good look at the reflectance equation we find that the diffuse $\Large{k_d}$ and specular $\Large{k_s}$ term of the BRDF are independent from each other and we can split the integral in two. 


$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi}) L_i(p, \omega_i)n \cdot \omega_i d \omega_i + \int\limits_{\Omega} (\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$ 

By splitting the integral in two parts we can focus on both the diffuse and specular term individually; the focus of these notes being on the diffuse integral. 

Taking a closer look at the diffuse integral we find that the diffuse lambert term is a constant term (the color **c**, the refraction ratio $\Large{k_d}$, and $\Large{\pi}$ are constant over the integral) and not dependent on any of the integral variables. Given this, we can move the constant term out of the diffuse integral. 

$\LARGE{L_o(p, \omega_o) = k_d \frac{c}{\pi} \int\limits_{\Omega} L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$ 
This gives us an integral that only depends on $\Large{\omega_i}$ (assuming $\Large{p}$ is at the center of the environment map). With this knowledge, we can calculate or *pre-compute* a new cubemap that stores in each sample direction (or texel) $\Large{\omega_o}$ the diffuse integral's result by **convolution**.

Convolution is apply some computation to each entry in a data set considering all other entries in the data set; the data set being the scene's radiance or environment map. Thus for every sample direction in the cubemap, we take all other sample directions over the hemisphere $\Large{\Omega}$ and averaging their radiance. The hemisphere we build the sample directions $\Large{\omega_i}$ from is oriented towards the output $\Large{\omega_o}$ sample direction we're convoluting. 

![[Pasted image 20251215123144.png]]

