
Blending in OpenGL is known as a technique to implement transparency within objects. Transparency is all about objects or parts of them not having a solid color, but having a combination of colors from the objects itself and any other object behind it with varying intensity. A colored glass window is a transparent object; the glass has a color o its own, but the resulting color contains the colors of all the objects behind the glass as well. This is also where the name blending comes from, since we blend several pixel colors (from different objects) to a single color. Transparency thus allows us to see through objects. 

Transparent objects can be completely transparent or partially transparent. The amount of transparency of an object is defined by its color's alpha value. The alpha color is the 4th component of the color vector that you've probably seen quite often now. We have previously specified this as 1.0 meaning 0 transparency. An alpha value of 0.0 would result in the object having complete transparency. An alpha value of 0.5 tells us the object's color consists of 50% of its own color, and 50% of the color of the objects behind it. 

The textures we've used so far all consisted of 3 color components: red, green, and blue, but some textures also have an embedded alpha value channel that contains an alpha value per texel. This alpha value tells us exactly which parts of the texture have transparency and by how much. For example this window texture has an alpha value of value of 0.25 at its glass part and an alpha value of 0.0 at its corners. The glass part would normally be completely red, but since it has 75% transparency it largely shows the page's background through it, making it seem a lot less red. 

We'll soon be adding this windowed texture to the scene from the depth testing. But first we'll discuss an easier technique to implement transparency for pixels that are either fully transparent or fully opaque. 

**Discarding Fragments**

Some effects do not care about partial transparency, but either want to show something or nothing at all based on the color value of a texture. Think of grass, to create something like grass with little effort you generally paste a grass texture onto a 2d quad and place that quad into your scene. However, grass isn't exactly shaped like a 2d square so you only want to display some parts of the grass texture and ignore the others. 

The grass texture provided on LearnOpenGL is a texture where it is either fully opaque or fully transparent and no in-between. 

So when adding vegetation to a scene we don't want to see a square image of grass, but rather only show the actual grass and see through the rest of the image. We want to discard the fragments that show the transparent parts of the texture, not storing that fragment into the color buffer. 

Before we get into that we first need to learn how to load a transparent texture. To load textures with alpha values there's not much we need to change. `stb_image` automatically loads an image's alpha channel if it's available, but we do need to tell OpenGL our texture now uses an alpha channel in the texture generation procedure. 

`glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE                                                                     data);`

Also make sure that you retrieve all 4 color components of the texture in the fragment shader, not just the RGB components.

`void main()`
`{`
	`// FragColor = vec4(vec3(texture(texture1, TexCoords)), 1.0);`
	`FragColor = texture(texture1, TexCoords);`
`}`

Now that we know how to load transparent textures it's time to put it to the test by adding several of these leaves of grass throughout the basic scene. 

We create a small vector array where we add several `glm::vec3` vectors to represent the location of the grass leaves. 

`vector<glm::vec3> vegetation;`
`vegetation.push_back(glm::vec3(-1.5f, 0.0f, -0.48f));   vegetation.push_back(glm::vec3( 1.5f, 0.0f, 0.51f)); vegetation.push_back(glm::vec3( 0.0f, 0.0f, 0.7f)); vegetation.push_back(glm::vec3(-0.3f, 0.0f, -2.3f)); vegetation.push_back(glm::vec3( 0.5f, 0.0f, -0.6f));`

Each of the grass objects is rendered as a single quad with the grass texture attached to it. It's not a perfect 3d representation of grass, but it's a lot more efficient than loading and rendering a large number of complex models. With a few tricks like adding randomized  rotations and scales you can get pretty convincing results with quads.

Because the grass texture is going to be displayed on a quad object we'll need to create another VAO again, fill the VBO, and set the appropriate vertex attribute pointers. Then after we've rendered the floor and the two cubes we're going to render the grass leaves. 

`glBindVertexArray(vegetationVAO);`
`glBindTexture(GL_TEXTURE_2D, grassTexture);`
`for(unsigned int i = 0; i < vegetation.size(); i++)`
`{`
	`model = glm::ma4(1.0f);`
	`model = glm::translate(model, vegetation[i]);`
	`shader.setMat4("model", model);`
	`glDrawArrays(GL_TRANGLES, 0, 6);`
`}`

If you run the app it should look like this:

![[Pasted image 20250602111320.png]]

This happens because OpenGL by default does not know what to do with alpha values, nor when to discard them. We have to manually do this ourselves. This is quite easy thanks to the use of shaders. GLSL gives us the discard command that once called ensures the fragment will not be further processed and thus not end up into the color buffer. Thanks to this command we can check whether a fragment has an alpha value below a certain threshold and if so, discard the fragment as if it had never been processed.

`#version 330 core`
`out vec4 FragColor;`

`in vec2 TexCoords;`

`uniform sampler2D texture1;`

`void main()`
`{`
	`vec4 texColor = texture(texture1, TexCoords);`
	`if(texColor.a < 0.1)`
		`discard;`
	`FragColor = texColor;`
`}`

Here we check if the sampled texture color contains an alpha value lower than a threshold of 0.1 and if so, discard the fragment. This fragment shader ensures us that it only renders fragments that are not (almost) completely transparent. Now it should look like this.

![[Pasted image 20250602113135.png]]

Note that when sampling textures at their borders, OpenGL interpolates the border values with the next repeated value of the texture (because we set its wrapping parameters to GL_REPEAT by default). This is usually okay, but since we're using transparent values, the top of the texture image gets its transparent value interpolated with the border's solid color value. The result is then a lightly transparent colored border you may see wrapped around your textured quad. To prevent this, set the texture wrapping method to `GL_CLAMP_TO_EDGE` whenever you use alpha textures that you don't want to repeat. 

`glTexParametersi(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);`
`glTexParametersi(GL_TEXTURE_@D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);`

**Blending**

While discarding fragments is great and all, it doesn't give us the flexibility to render transparent images; we either render the fragment or completely discard it. To render images with different levels of transparency we have to enable blending. Like most of OpenGL's functionality we can enable blending by enabling `GL_BLEND`.

`glEnable(GL_BLEND);`

Now that we've enabled blending we need to tell OpenGL how it should actually blend. 

Blending in OpenGL happens with the following equation: 

![[Pasted image 20250602114435.png]]

C source: the source color vector. This is the color output of the fragment shader.
C destination: the destination color vector. This is the color vector that is currently stored in the color buffer. 
F source: the destination color vector. This is the color vector that is currently stored in the color buffer.
F source: the source factor value. Sets the impact of the alpha value on the source color.
F destination: the destination factor value. Sets the impact of the alpha value on the destination color. 

After the fragment shader has run and all the tests have passed. this blend equation is let loose on the fragments color output and with whatever is currently in the color buffer. The source and destination colors will be automatically be set by OpenGL, but source and destination factor can be set to a value of our choosing. 

![[Pasted image 20250602115105.png]]

We have two squares where we want to draw the transparent green square on top of the red square. The red square will be the destination color and thus should be first in the color buffer and we are now going to draw the green square over the red square. 

The question then arises: what do we set the factor values to? Well, we at least want to multiply the green square with its alpha value so we want to set the F source equal to the alpha value of the source color vector which is 0.6. Then it makes sense to let the destination square have a contribution equal to the remainder of the alpha value. If the green square contributes 60% to the final color we want the red square to contribute 40% of the final color e.g. 1.0 - 0.6 = 0.4. So we set F destination equal to one minus the alpha value of the source color vector. The equation thus becomes:

![[Pasted image 20250602120056.png]]

The result is that the combined square fragments contain a color that is 60% green and 40% red. 

The resulting color is then stored in the color buffer, replacing the previous color. 

So this is great and all, but how do we actually tell OpenGL to use factors like that? Well it just so happens that there is a function called `glBlendFunc`.

The `glBlendFunc(GLenum sfactor, GLenum dfactor)` function expects two parameters that set the option for the source and destination factor. OpenGL defined quite a few options for us to set of which we'll list the most common options below. Note that the constant color vector C constant can be separately set via the `glBlendColor` function. 

| Option                        | Value                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------- |
| `GL_ZERO`                     | Factor is equal to 0.                                                                 |
| `GL_ONE`                      | Factor is equal to 1.                                                                 |
| `GL_SRC_COLOR`                | Factor is equal to the source color vector C¯source.                                  |
| `GL_ONE_MINUS_SRC_COLOR`      | Factor is equal to 1 minus the source color vector: 1−C¯source.                       |
| `GL_DST_COLOR`                | Factor is equal to the destination color vector C¯destination.                        |
| `GL_ONE_MINUS_DST_COLOR`      | Factor is equal to 1 minus the destination color vector: 1−C¯destination.             |
| `GL_SRC_ALPHA`                | Factor is equal to the alpha component of the source color vector C¯source.           |
| `GL_ONE_MINUS_SRC_ALPHA`      | Factor is equal to 1−alpha of the source color vector C¯source.                       |
| `GL_DST_ALPHA`                | Factor is equal to the alpha component of the destination color vector C¯destination. |
| `GL_ONE_MINUS_DST_ALPHA`      | Factor is equal to 1−alpha of the destination color vector C¯destination.             |
| `GL_CONSTANT_COLOR`           | Factor is equal to the constant color vector C¯constant.                              |
| `GL_ONE_MINUS_CONSTANT_COLOR` | Factor is equal to 1 - the constant color vector C¯constant.                          |
| `GL_CONSTANT_ALPHA`           | Factor is equal to the alpha component of the constant color vector C¯constant.       |

To get the blending result of our little two square example, we want to take the alpha of the source color vector for the source factor and 1 - alpha of the same color vector for the destination factor. This translates to `glBlendFunc` as follows:

`glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);`

It is also possible to set different options for the RGB and alpha channel individually using `glBlenFuncSeperate`

`glBlendFuncSeperate(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA, GL_ONE, GL_ZERO);`

The function sets the RGB components as we've set them previously, but only lets the resulting alpha component be influenced by the source's alpha value. 

OpenGL gives us even more flexibility by allowing us to change the operator between the source and destination part of the equation. Right now, the source and destination components  are added together, but we could also subtract them if we want. `glBlendEquation(GLenum mode)` allows us to set this operation and has 5 possible options. 

- `GL_FUNC_ADD`: the default, adds both colors to each other: C result = Source + Destination.
- `GL_FUNC_SUBTRACT`: subtracts both colors from each other: C result = Source - Destination. 
- `GL_FUNC_REVERSE_SUBTRACT`: subtracts both colors, but reverses order: C result = Destination - Source.
- `GL_MIN`: takes the component-wise minimum of both colors: C result = min(Destination, Source).
- `GL_MAX`: takes the component-wise maximum of both colors: C result = max(Destination, Source).

Usually we can simply omit a call to `glBlendEquation` because `GL_FUNC_ADD` is the preferred blending equation for most operations, but if you're really trying your best to break the mainstream circuit any of the other equation could suit your needs. 

**Rendering Transparent Textures**

Now that we know how OpenGL works with regards to blending it's time to put our knowledge to the test by adding several transparent widows. We'll be using the same scene as in the beginning, but instead of rendering a grass texture we're now going to use the transparent window texture. 

First, during initialization we enable blending and set the appropriate blending functions. 

`glEnable(GL_BLEND);`
`glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);`

Since we enabled blending there is no need to discard fragments so we'll reset the fragment shader to its original version.

`#version 330 core`
`out vec4 FragColor;`

`in vec2 TexCoords;`

`void main()`
`{`
	`FragColor = texture(texture1, TexCoords);`
`}`

This time (whenever OpenGL renders a fragment) it combines the current fragment's color with the fragment color currently in the color buffer based on the alpha value of `FragColor`. Since the glass part of the window texture is transparent we should be able to see the rest of the scene by looking through the window. 

If you take a closer look however, you may notice something is off. The transparent parts of the front window are occluding the windows in the background,. Why is this happening?

The reason for this is that depth testing works a bit tricky combined with blending. When writing to the depth buffer, the depth test does not care if the fragment has transparency or not, so the transparent parts are written to the depth buffer as any other value. The result is that the background windows are tested on depth as any other opaque object would be, ignoring transparency. Even though the transparent part should show the windows, the depth discards them. 

So we cannot simply render the windows however we want and expect the depth buffer to solve all our issues for us; this is also where blending gets a little nasty. To make sure the windows show the windows behind them, we have to draw the windows in the background first. This means we have to manually sort the windows from furthest to nearest and draw them accordingly ourselves. 

Note that with fully transparent objects like the grass leaves we have the option to discard the transparent fragments instead of blending them, saving us a few of these headaches (no depth issues). 

**Don't Break the Order**

To make blending work for multiple objects we have to draw the most distant object first and the closest object last. The normal non-blended objects can still be drawn as normal using the depth buffer so they don't have to be sorted. We do have to make sure they are drawn first before drawing the (sorted) transparent objects. When drawing a scene with non-transparent and transparent objects the general outline is usually as follows.

1. Draw all opaque objects first. 
2. Sort all the transparent objects. 
3. Draw all the transparent objects in sorted order.

One way of sorting the transparent objects is to retrieve the distance of an object from the viewer's perspective. This can be achieved by taking the distance between the camera's position vector and the object's position vector. We then store this distance together with the corresponding position vector in a map data structure from the STL library. A map automatically sorts its value based on its keys, so once we've added all positions with their distance as the key they're automatically sorted on their distance value.

`std::map<float, glm::vec3> sorted;`
`for (unsigned int i = 0; i < windows.size(); i++)`
`{`
	`float distance = glm::length(camera.Position - windows[i]);`
	`sorted[distance] = windows[i];`
`}`

The result is a sorted container object that stores each of the window positions based on their distance key value from lowest to highest distance.

Then, this time when rendering, we take each of the map's values in reverse order (from farthest to nearest) and then draw the corresponding windows in correct order. 

`for(std::map<float,glm::vec3>::reverse_iterator it = sorted.rbegin(); it != sorted.rend(); ++it)`
`{`
	`model = glm::mat4(1.0f);`
	`model = glm::translate(model, it->second);`
	`shader.setMat4("model", model);`
	`glDrawArray(GL_TRIANGLES, 0, 6);`
`}`

We take a reverse iterator from the map to iterate through each of the items in reverse order and then translate each window quad to the corresponding window position. This relatively simple approach to sorting transparent object fixes the previous problem and now the scene looks like this. 

While this approach of sorting the objects by their distance works well for this specific scenario, it doesn't take rotations, scaling or any other transformation into account and weirdly shaped objects need a different metric then simply a position vector. 

