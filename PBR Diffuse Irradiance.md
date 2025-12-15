
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

This pre-computed cubemap, that for each sample direction $\Large{\omega_o}$ stores the integral result, can be though of as the pre-computed sum of all indirect diffuse light of the scene hitting some surface aligned along direction $\Large{\omega_o}$. Such a cubemap is known as an **irradiance map** seeing as the convoluted effectively allows us to directly sample the scene's (pre-computed) irradiance from any direction $\Large{\omega_o}$. 

The radiance equation also depends on a position $\Large{p}$, which we've assumed to be at the center of the irradiance map. This does mean all diffuse indirect light must come from a single environment map which may break the illusion of reality (especially indoors). Render engines solve this by placing **reflection probes** all over the scene where each reflection probes calculates its own irradiance map of its surroundings. This way, the irradiance (and radiance) at position $\Large{p}$ is the interpolated irradiance between its closest reflection probes. For now, we assume we always sample the environment map from its center. 

Below is an example of a cubemap environment map and its resulting irradiance map (courtesy of [wave engine](http://www.indiedb.com/features/using-image-based-lighting-ibl)), averaging the scene's radiance for every direction $\Large{\omega_o}$. 

![[Pasted image 20251215124413.png]]

By storing the convoluted result in each cubemap texel (in the direction of $\Large{\omega_o}$), the irradiance map displays somewhat like an average color or lighting display of the environment. Sampling any direction from this environment map will give us the scene's irradiance in that particular direction. 

**PBR and HDR**

We've briefly touched upon it in the [previous](https://learnopengl.com/PBR/Lighting) notes: taking the high dynamic range of your scene's lighting into account in a PBR pipeline is incredibly important. As PBR bases most of its inputs on real physical properties and measurements it makes sense to closely match the incoming light values to their physical equivalents. Whether we make educated guesses on each light's radiant flux or use their [direct physical equivalent](https://en.wikipedia.org/wiki/Lumen_\(unit\)), the difference between a simple light bulb or the sun is significant either way. Without working in an [HDR](https://learnopengl.com/Advanced-Lighting/HDR) render environment it's impossible to correctly specify each light's relative intensity. 

So, PBR and HDR go hand in hand, but how does it relate to image based lighting? We've seen in the previous notes that it's relatively easy to get PBR working in HDR. However, seeing as for image based lighting we base the environment's indirect light intensity on the color values of an environment cubemap we need some way to store the lighting's high dynamic range into an environment map. 

The environment maps we've been using so far as cubemaps (used as [skyboxes](https://learnopengl.com/Advanced-OpenGL/Cubemaps) for instance) are in low dynamic range (LDR). We directly used their color values from the individual face images, ranged between $0.0$ and $1.0$, and processed them as is. While this may work fine for visual output, when taking them as physical input parameters it's not going to work. 

**The Radiance HDR File Format**

Enter the radiance file format. The radiance file format (with the `.hdr` extension) stores a full cubemap with all 6 faces as floating point data. This allows us to specify color values outside the $0.0$ and $1.0$ range to give lights their correct color intensities. The file format also uses a clever trick to store each floating point value, not as a 32 bit value per channel, but 8 bits per channel using the color's alpha channel as an exponent (this does come with a loss of precision). This works quite well, but requires the parsing program to re-convert each color to their floating point equivalent.

There are quite a few radiance HDR environment maps freely available form sources like [sIBL archive](http://www.hdrlabs.com/sibl/archive.html) of which you can see an example below. 

![[Pasted image 20251215130835.png]]

This may not be exactly what you were expecting, as the image appears distorted and doesn't show any of the 6 individual cubemap faces of environment maps we've seen before. This environment map is projected from a sphere onto a flat plane such that we can more easily store the environment into a single image known as an **equirectangular map**. This does come with a small caveat as most of the visual resolution is stored in the horizontal view direction, while less is preserved in the bottom and top directions. In most cases this is a decent compromise as with almost any renderer you'll find most of the interesting lighting and surroundings in the horizontal viewing directions. 

**HDR and stb_image.h**

Loading radiance HDR images directly requires some knowledge of the [file format](http://radsite.lbl.gov/radiance/refer/Notes/picture_format.html) which isn't too difficult, but cumbersome nonetheless. Lucky for us, the popular one header library [stb_image.h](https://github.com/nothings/stb/blob/master/stb_image.h) supports loading radiance HDR images directly as an array of floating point values which perfectly fits our needs. With `stb_image` added to your project, loading an HDR image is now as simple as follows. 

```
#include "stb_image.h"

[...]

stbi_set_flip_vertically_on_load(true);
in width, height, nrComponents; 
float *data = stbi_loadf("newport_loft.hdr", &width, &height, &nrComponents, 0);
unigned int hdrTexture;
if (data)
{
	glGenTextures(1, &hdrTexture);
	glBindTexture(GL_TEXTURE_2D, hdrTexture);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, width, height, 0, GL_RGB, GL_FLOAT,      data);
	
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	
	stbi_image_free(data);
}
else
{
	std::cout << "Failed to load HDR image." << std::endl;
}
```

`stb_image.h` automatically maps the HDR values to a list of floating point values: 32 bits per channel and 3 channels per color by default. This is all we need to store the equirectangular HDR environment map into a 2D floating point texture. 

**From Equirectangular to Cubemap**

It is possible to use the equirectangular map directly for environment lookups, but these operations can be relatively expensive in which case a direct cubemap sample is more performant. Therefore, in these notes we'll first convert the equirectangular image to a cubemap for further processing. Note that in the process we also show how to sample an equirectangular map as if it was a 3D environment map in which case you're free to pick whichever solution you prefer. 

To convert an equirectangular image into a cubemap we need to render a (unit) cube and project the equirectangular map on all of the cube's faces from the inside and take 6 images of each of the cube's sides as a cubemap face. The vertex shader