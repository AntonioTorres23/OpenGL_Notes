
IBL, or **image based lighting**, is a collection of techniques to light objects, not by direct analytical lights as in the [previous](https://learnopengl.com/PBR/Lighting) notes, but by treating the surrounding environment as one big light source. This is generally accomplished by manipulating a cubemap environment map (taken from the real world or generated from a 3D scene) such that we can directly use it in our lighting equations: treating each cubemap texel as a light emitter. This way we can effectively capture an environment's global lighting and general feel, giving objects a better sense of *belonging* in their environment. 

As image based lighting algorithms capture the lighting of some (global) environment, its input is considered a more precise form of ambient lighting, even a crude approximation of global illumination. This makes IBL interesting for PBR as objects look significantly more physically accurate when we take the environments lighting into account. 

To start introducing IBL into our system let's take a quick look at the reflectance equation. 

$\LARGE{L_o(p, \omega_o) = \int\limits_{\Omega} (k_d \frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})L_i(p, \omega_i)n \cdot \omega_i d \omega_i}$    

As described before, our main goal is to solve the integral of all incoming light directions $\Large{\omega_i}$ over the hemisphere $\Large{\Omega}$. Solving the integral in the previous notes was easy as we knew beforehand the exact few light directions $\Large{\omega_i}$ that contributed to the integral. This time however 