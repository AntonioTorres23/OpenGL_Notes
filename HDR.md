
Brightness and color values, by default, are clamped between $0.0$ and $1.0$ when stored into a framebuffer. This, at first seemingly innocent, statement caused us to always specify light and color values somewhere in this range, trying to make them fit into the scene. This works ok and gives decent results, but what happens if we walk in a really bright area with multiple bright light sources that as a total sum exceed 1.0? The answer is that all fragments that have a brightness or color sum over $1.0$ gets clamped to $1.0$, which isn't pretty to look at. 

![[Pasted image 20251114164910.png]]

Due to a large number of fragments' color value getting clamped to $1.0$, each of the bright fragments have the exact same white color value in large regions, losing a significant amount of detail and giving it a fake look. 

A solution to this problem would be to reduce the strength of the light sources and ensure no area of fragments in your scene ends up brighter than ; this is not a good solution as this forces you to use unrealistic lighting parameters. A better approach is to allow color values to temporarily exceed $1.0$ and transform them back to the original range of $0.0$ and $1.0$ as a final step, but without losing detail. 

Monitors (non-HDR) are limited to display colors in the range of $0.0$ and $1.0$, but there is no such limitation in lighting equations. By allowing fragment colors to exceed $1.0$ we have a much higher range of color values available to work in known as **High Dynamic Range** (HDR). With high dynamic range, bright things can be really bright, dark things can be really dark, and details can be seen in both. 

High dynamic range was originally only used for photography where a photographer takes multiple pictures of the same scene with varying exposure levels, capturing a large range of color values. Combining these forms a HDR image where a range of details are visible based on the combined exposure levels, or a specific exposure it is viewed with. For instance, the following image (credits to Colin Smith) shows a lot of detail at brightly lit regions with a low exposure (look at the window), but these details are gone with a high exposure. However, a high exposure now reveals a great amount of detail at darker regions that weren't previously visible. 

![[Pasted image 20251117150757.png]]

This is also very similar to how the human eye works and the basis of high dynamic range rendering. When there is little light, the human eye adapts itself so the darker parts become more visible and similarly for bright areas. It's like the human eye has an automatic exposure slider on the scene's brightness.

High dynamic range rendering works a bit like that. We allow for a much larger range of color values to render to, collection a large range of dark and bright details of a scene, and at the end we transform all HDR values back to **low dynamic range** (LDR) of $[0.0, 1.0]$. This process of converting HRD values to LDR values is called **tone mapping** and a large collection of tone mapping algorithms exist that aim to preserve most HDR details during the conversion process. These tone mapping algorithms often involve an exposure parameter that selectively favors dark or bright regions. 

When it comes to real-time rendering, high dynamic range allows us to not only exceed the LDR range of $[0.0, 1.0]$ and preserve more detail, but it also gives us the ability to specify a light source's intensity by their *real* intensities. For instance, the sun has a much higher intensity than something like a flashlight so why not configure the sun as such (e.g. a diffuse brightness of $100.0$). This allows us to more properly configure a scene's lighting with more realistic lighting parameters, something that wouldn't be possible with LDR rendering as they'd then directly get clamped to $1.0$. 

As (non-HDR) monitors only display colors in the range between $0.0$ and $1.0$ we do need to transform the currently dynamic range of color values back to the monitor's range. Simply re-transforming the colors back with a simple average wouldn't do us much good as brighter areas then become a lot more dominant. What we can do, is use different equations and/or curves to transform the HDR values back to LDR that give us complete control over the scene's brightness. This is the process earlier denoted as tone mapping and the final step of HDR rendering. 

**Floating Point Framebuffers**

To implement high dynamic range rendering we need some way to prevent color values getting clamped after each fragment shader run. When framebuffers use a normal fixed-point color format (like `GL_RGB`) as their color buffer's internal format, OpenGL automatically clamps the values between $0.0$ and $1.0$ before storing them in the framebuffer. This operation holds for most types of framebuffer formats, except for floating point formats. 

When the internal format of a framebuffer's color buffer is specified as `GL_RGB16F`, `GL_RGBA16F`, `GL_RGB32F`, `GL_RGBA32F` the framebuffer is known as a **floating point framebuffer** that can store floating point values outside default range of $0.0$ and $1.0$. This is perfect for rendering in high dynamic range!

To create a floating point framebuffer the only thing we need to change is its color buffer's internal format parameter.

```
glBindTexture(GL_TEXTURE_2D, colorBuffer);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA16F, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGBA, GL_FLOAT, NULL);
```

The default framebuffer of OpenGL (by default) only takes up 8 bits per color component. With a floating point framebuffer with 32 bits per color components (when USING `GL_RGBA32F` or `GL_RGB32F`) we're using 4 times more memory for storing color values. As 32 bit isn't really necessary (unless you need a high level of precision) using `GL_RGBA16F` will suffice. 

With a floating point color buffer attached to a framebuffer we can now render the scene into this framebuffer knowing color values won't get clamped between $0.0$ and $1.0$. In this note's example demo we first render a lit scene into the floating point framebuffer and then display the framebuffer's color buffer on a screen-filled quad; it'll look a bit like this.

```
glBindFramebuffer(GL_FRAMEBUFFER, hdrFBO);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
	// [...] render (lit) scene
glBindFramebuffer(GL_FRAMEBUFFER, 0);

// now render hdr color bufferto 2D screen-filling quad with tone mapping shader
hdrShader.use();
glActivateTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, hdrColorBufferTexture);
RenderQuad();
```

Here a scene's color values are filled into a point color buffer which can contain any arbitrary color value, possibly exceeding $1.0$. For these notes, a simple demo scene was created with a large stretched cube acting as a tunnel with four point lights, one being extremely bright positioned at the tunnel's end. 

```
std::vector<glm::vec3> lightColors;
lightColors.push_back(glm::vec3(200.0f, 200.0f, 200.0f));
lightColors.push_back(glm::vec3(0.1f, 0.0f, 0.0f));
lightColors.push_back(glm::vec3(0.0f, 0.0f, 0.2f));
lightColors.push_back(glm::vec3(0.0f, 0.1f, 0.0f));
```

Rendering to a floating point framebuffer is exactly the same as we would normally render into a framebuffer. What is new is `hdrShader`'s fragment shader that renders the final 2D quad with the floating point color buffer texture attached. Let's first define a simple pass-through fragment shader. 

```
#version 330 core
out vec4 FragColor; 

in vec2 TexCoords;

uniform smapler2D hdrBuffer;

void main()
{
	vec3 hdrColor = texture(hdrBuffer, TexCoords).rgb;
	FragColor = vec4(hdrColor, 1.0)
}
```

Here we directly sample the floating point color buffer and use its color value as the fragment shader's output. However, as the 2D quad's output is directly rendered into the default framebuffer, all the fragment shader's output values will still end up clamped between $0.0$ and $1.0$ even though we have several values in the floating point color texture exceeding $1.0$. 

![[Pasted image 20251118170225.png]]

It becomes clear that the intense light values at the end of the tunnel are clamped to $1.0$ as a large portion of it is completely white, effectively losing all lighting details in the process. 








































