
We've been using 2D textures for a while now, but there are more texture types we haven't explored yet and in this chapter we'll discuss a texture type that is a combination of multiple textures mapped into one: a **cubemap**. 

A cube map is a texture that contains 6 individual 2D textures that each form one side of a cube: a textured cube. You may be wondering what the point is of such cube? Why bother combining 6 individual textures into a single entity instead of just using 6 individual textures? Well, cube maps have the useful property that they can be indexed/sampled using a direction vector. Imagine we have a 1x1x1 unit cube with the origin of a direction vector residing at its center. Sampling a texture value from the cube map with an orange direction vector looks a bit like this. 

![[Pasted image 20250609112403.png]]

The magnitude of the direction vector doesn't matter. As long as a direction is supplied, OpenGL retrieves the corresponding texels that the direction hits (eventually) and returns the properly sampled texture value. 

If we imagine we have a cube shape that we attach such a cubemap to, this direction vector would be similar to the (interpolated) local vertex position of the cube. This way we can sample the cubemap using the cube's actual position vectors as long as the cube is centered on the origin. We thus consider all vertex positions of the cube to be its texture coordinates when sampling a cube map. The result is a texture coordinate that accesses the proper face texture of the cubemap. The result is a texture coordinate that accesses the proper individual face texture of the cubemap. 

**Creating a Cubemap**

A cubemap is a texture like any other texture, so to create one we generate a texture and bind it to the proper texture target before we do any further texture operations. This time binding it to `GL_TEXTURE_CUBE_MAP:`

`unsigned int textureID;`
`glGenTextures(1, &textureID);`
`glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);`

Because a cubemap contains 6 textures, one for each face, we have to call `glTexImage2D` six times with their parameters set similarly to the previous chapters. This time however, we have to set the texture target parameter to match a specific face of the cubemap, telling OpenGL which side of the cubemap we're creating a texture for. This means we have to call `glTexImage2D` once for each face of the cubemap.

Since we have 6 faces OpenGL gives us 6 special texture targets for targeting a face of the cubemap.

|Texture target|Orientation|
|---|---|
|`GL_TEXTURE_CUBE_MAP_POSITIVE_X`|Right|
|`GL_TEXTURE_CUBE_MAP_NEGATIVE_X`|Left|
|`GL_TEXTURE_CUBE_MAP_POSITIVE_Y`|Top|
|`GL_TEXTURE_CUBE_MAP_NEGATIVE_Y`|Bottom|
|`GL_TEXTURE_CUBE_MAP_POSITIVE_Z`|Back|
|`GL_TEXTURE_CUBE_MAP_NEGATIVE_Z`|Front|

Like many of OpenGL's `enums`, their behind-the-scenes int value is linearly incremented, so if we were to have an array or vector of texture locations we could loop over them by starting with `GL_TEXTURE_CUBE_MAP_POSITVE_X` and incrementing the `enum` by 1 each iteration, effectively looping through all the texture targets.

`int width, height, nrChannels;`
`unsigned char *data;`
`for(unsigned int i = 0; i < textures_faces.size(); i++)`
`{`
	`data = stbi_load(texture_faces[i].c_str(), &width, &height, &nrChannels, 0);`
	`glTexImage2D(`
		`GL_TEXTURE_CUBE_MAP_POSITIVE_X + i,`
		`0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data` 
	`);`
`}`

Here we have a vector called `textures_faces` that contain the locations of all the textures required for the cubemap in the order as given in the table. This generates a texture for each face of the currently bound cubemap. 

Because a cubemap is a texture like any other texture, we will also specify its wrapping and filtering methods. 

`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);`
`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);`
`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);`
`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);`
`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);`

Don't be scared by the `GL_TEXTURE_MAP_R`, this simply sets the wrapping methods for the texture's R coordinate which corresponds to the texture's 3rd dimension (like z for positions). We set the wrapping method to `GL_CLAMP_TO_EDGE` since texture coordinates that are exactly between two faces may not hit an exact face (due to some hardware limitations) so by using `GL_CLAMP_TO_EDGE` OpenGL always returns their edge values whenever we sample between faces. 

Then before drawing the objects that will use the cubemap, we activate the corresponding texture unit and bind the cubemap before rendering; not much of a difference compared to normal 2D textures. 

Within the fragment shader we also have to use a different sampler of the type `samplerCube` that we sample from using the texture function, but this time using a vec3 direction vector instead of vec2. An example of fragment shader using a cubemap looks like this. 

`in vec3 textureDir; // direction vector representing a 3D texture coordinate`
`uniform samplerCube cubemap; // cubemap texture sampler`

`void main()`
`{`
	`FragColor = texture(cubemap, textureDir);`
`}`

This is used to develop something in game development called a skybox. 

**Skybox**

A skybox is a large cube that encompasses the entire scene and contains 6 images of a surrounding environment, giving the player the illusion that the environment he's in is actually much larger than it actually is. Some examples of skyboxes used in videogames are images of mountains, clouds, or of a starry night sky. 

Skyboxes suit cubemaps perfectly, we have a cube that has 6 faces and needs to be textured per face. In the previous image they used several images of a night sky to give the illusion the player is in some large universe while he's actually inside a tiny box. 

There are usually enough resources online where you could find skyboxes like that. These skybox images usually have the following pattern. 

![[Pasted image 20250611142819.png]]


If you would fold these 6 sides into a cube you'd get the completely textured cube that simulates a large landscape. Some resources provide the skybox in a format like that in which case you'd have to manually extract the 6 face images, but in most cases they're provided as 7 single texture images.

This skybox is what we'll be using in this tutorial and can be downloaded [here](https://learnopengl.com/img/textures/skybox.zip). 

**Loading a Skybox**

Since a skybox is by itself just a cubemap, loading a skybox isn't too different from what we've seen at the start of this chapter. To load the skybox we're going to use the following function that accepts a vector of 6 texture locations.

`unsigned int loadCubemap(vector<std::string> faces)`
`{`
	`unsigned int textureID;`
	`glGenTextures(1, &textureID);`
	`glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);`

	`int width, height, nrChannels;`
	`for(unsigned int i = 0; i < faces.size(); i++)`
	`{`
		`unsigned char *data = stbi_load(faces[i].c_str(), &width, &height,                  &nrChannels, 0);`
		`if(data)`
		`{`
			`glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGB, width,                  height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);`
			`stbi_image_free(data);`
		`}`
		`else`
		`{`
			`std::cout << "Cubemap tex failed to load at path: " << faces[i] <<                 std::endl;`
			`stbi_image_free(data);`
		`}`
	`}`

	`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);`
	`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);`
	`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);`
	`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);`
	`glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);`

	`return textureID;`
`}`


The function itself shouldn't be too surprising. It is basically all the cubemap code we've seen in the previous section, but combined in a single manageable function. 

Now, before we call this function we'll load the appropriate texture paths in a vector in the order as specified by the cubemap `enums`.

`vector<std::string> faces;`
`{`
	`"right.jpg",`
	`"left.jpg",`
	`"top.jpg",`
	`"bottom.jpg,"`
	`"front.jpg",`
	`"back.jpg"`
`};`
`unsigned int cubemapTexture = loadCubemap(faces);`

We loaded the skybox as a cubemap with `cubemapTexture` as its id. We can now finally bind it to a cube to replace the clear color we've been using this whole time. 

**Displaying a Skybox**

Because a skybox is drawn on a cube we'll need another VAO, VBO, and a fresh set of vertices like any other 3D object. You can get its vertex data [here](https://learnopengl.com/code_viewer.php?code=advanced/cubemaps_skybox_data). 

A cubemap used to texture a 3D cube can be sampled using the local positions of the cube as its textured coordinates. When a cube is centered on the origin (0,0,0) each of its position vectors is also a direction vector from the origin. This direction vector is exactly what we need to get the corresponding texture value at that specific cube's position. For this reason we only need to supply position vectors and don't need texture coordinates. 

To render the skybox we'll need a new set of shader which aren't too complicated. Because we only have one vertex attribute the vertex shader is quite simple.

`#version 330 core`
`layout (location = 0) in vec3 aPos;`

`out vec3 TexCoords;`

`uniform mat4 projection;`
`uniform mat4 view;`

`void main()`
`{`
	`TexCoords = aPos;`
	`gl_Position = projection * view * vec4(aPos, 1.0);`
`}`

The interesting part of this shader is that we set the incoming local position vector as the outcoming texture coordinate for (interpolated) use in the fragment shader. The fragment shader then takes these input to sample a `samplerCube`.

`#version 330 core`
`out vec4 FragColor;`

`in vec3 TexCoords;`

`uniform samplerCube skybox;`

`void main()`
`{`
	`FragColor = texture(skybox, TexCoords);`
`}`


The fragment shader is relatively straightforward. We take the vertex attribute's interpolated position vector as the texture's direction vector and use it to sample the texture values from the cubemap. 

Rendering the skybox is easy now that we have a cubemap texture, we simply bind the cubemap texture and the skybox sampler is automatically filled with the skybox cubemap. To draw the skybox we're going to draw it as the first object in the scene and disable depth writing. This way the skybox will always be drawn at the background of other objects since the unit cube is most likely smaller than the rest of the scene. 

`glDepthMask(GL_FALSE);`
`skyboxShader.use();`
`// ... set view and projection matrix`
`glBindVertexArray(skyboxVAO);`
`glBindTexture(GL_TEXTURE_CUBE_MAP, cubemapTexture);`
`glDrawArrays(GL_TRIANGLES, 0, 36);`
`glDepthMask(GL_TRUE);`
`// ... draw rest of the scene`


If you run this now you will get difficulties though. We want the skybox to be centered around the player so that no matter how far the player moves, the skybox won't get any closer, giving the impression the surrounding environment is extremely large. The current view matrix however transforms all skybox's positions by rotating, scaling, and translating them, so if the player moves, the cubemap moves as well. We want to remove the translation part of the view matrix so only rotation will affect the skybox's position vectors. 

We can remove the translation section of transformation matrices by taking the upper-level 3x3 matrix of the 4x4 matrix. We can achieve this by converting the view matrix to 3x3 matrix (removing translation) and converting it back to a 4x4 matrix.

`glm::mat4 view = glm::mat4(glm::mat3(camera.GetViewMatrix()));`

This removes any translation, but keeps all rotation transformations so the user can still look around the scene. 

The result is a scene that instantly looks enormous due to our skybox. If you'd fly around the basic container you'd immediately get a sense of scale. which dramatically improves the scene, The result looks something like this.  

![[Pasted image 20250613102653.png]]

Try experimenting with different skyboxes and see how 