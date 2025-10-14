
Say you have a scene where you're drawing a lot of models where most of these models contain the same set of vertex data, but with different world transformations. Think of a scene filled with grass leaves: each grass leave is a small model that consists of only a few triangles. You'll probably want to draw quite a few of them and your scene may end up with thousands or maybe tens of thousands of grass leaves that you need to render each frame. Because each leaf is only a few triangles, the leaf is rendered almost instantly. However, the thousands of render calls you'll have to make will drastically reduce performance. 

If we were to actually render such a large amount of objects it will look a bit like this in code. 

```
for (unsigned int i = 0; i < amount_of_models_to_draw; i++)
{
	DoSomePreperations(); // bind VAO, bind textures, set uniforms etc.
	glDrawArrays(GL_TRIANGLES, 0, amount_of_vertices);
}
```

When drawing many **instances** of your model like this you'll quickly reach a performance bottle neck because of the many draw calls. Compared to rendering the actual vertices, telling the GPU to render your vertex data with functions like `glDrawArrays` or `glDrawElements` eats up quite some performance since OpenGL must make necessary preparations before it can draw your vertex data (like telling the GPU which buffer to read data from, where to find vertex attributes, and all this over the relatively slow CPU to GPU bus). So even though rendering your vertices is super fast, giving your GPU the commands to render them isn't. 

It would be more convenient if we could send data over to the GPU once, and then tell OpenGL to draw multiple objects using this data with a single drawing call. Enter **instancing**. 

Instancing is a technique where we draw many (equal mesh data) objects at once with a single render call, saving all the CPU -> GPU communications each time we need to render an object. To render using instancing all we need to do is change the render calls `glDrawArrays` and `glDrawElements` to `glDrawArraysInstanced` and `glDrawElementsInstanced` respectively. These *instanced* versions of the classic rendering functions take an extra parameter called the **instancing count** that sets the number of instances we want to render. We sent all the required data to the GPU once, and then tell the GPU how it should draw all these instances with a single call. The GPU then renders all these instances without having to continually communicate with the CPU.

By itself this function is a bit useless. Rendering the same object a thousand times is of no use to use since each of the rendered objects is rendered exactly the same and thus also at the same location; we would only see one object! For this reason GLSL added another built-in variable in the vertex shader called `gl_InstanceID`. 

When drawing with one of the instanced rendering calls, `gl_InstancedID` is incremented for each instance being rendered starting from 0. If we were to render the 43th instance for example, `gl_InstanceID` would have the value 42 in the vertex shader. Having a unique value per instance means we could now for example, index into a large array of position values to position each instance at a different location in the world. 

To get a feel for instanced drawing we're going to demonstrate a simple example that renders a hundred 2D quads in normalized device coordinates with just one render call. We accomplish this by uniquely positioning each instanced quad by indexing a uniform array of 100 offset vectors. The result is a neatly organized grid of quads that fill the entire window. 

![[Pasted image 20251014152904.png]]

Each quad consists of 2 triangles with a total of 6 vertices. Each vertex contains a 2D Normalized Device Coordinate (NDC) position vector and a color vector. Below is the vertex data used for this example - the triangles are small enough to properly fit the screen when there's 100 of them. 

```
float quadVertices[] = 
{
	// positions   // colors 
	// 1st triangle
	// top left
	-0.05f, 0.05f, 1.0f, 0.0f, 0.0f, // red
	// bottom right
	0.05f, -0.05f, 0.0f, 1.0f, 0.0f, // green
	// bottom left   
	-0.05f, -0.05f, 0.0f, 0.0f, 1.0f, // blue
	// 2nd triangle
	// top left
	-0.05f, 0.05f,  1.0f, 0.0f, 0.0f, // red
	// bottom right
	0.05f, -0.05f,  0.0f, 1.0f, 0.0f, // green
	// top right
	0.05f, 0.05f,   0.0f, 0.0f, 1.0f  // purple
}
```

The quads are colored in the fragment shader that receives a color vector from the vertex shader and sets it as its output. 

```
#version 330 core
out vec4 FragColor; 

in vec3 fColor;

void main()
{
	FragColor = vec4(fColor, 1.0f);
}
```

Nothing new so far, but at the vertex shader it's starting to get interesting. 

```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;

out vec3 fColor;

uniform vec2 offsets[100];

void main()
{
	vec2 offset = offsets[gl_InstanceID];
	// set z coordinate to 0 because we are working in 2D 
	gl_Position = vec4(aPos + offset, 0.0, 1.0); 
	fColor = aColor;
}
```

Here we defined a uniform array called **offsets** that contain a total of 100 offset vectors. Within the vertex shader we retrieve an offset vector for each instance by indexing the **offsets** array using `gl_InstanceID`. If we now were to draw 100 quads with instanced drawing we'd get 100 quads location at different positions. 

We do need to actually set the offset positions that we calculate in a nested for-loop before we enter the render loop. 

```
glm::vec2 translations[100];
int index = 0;
float offset = 0.1f;
// increment for-loop argument (y) by 2
for(int y = -10; y < 10; y +=2)
{
	// increment for-loop argument (x) by 2
	for(int x = -10; x < 10; x += 2)
	{
		glm::vec2 translation;
		// for-loop argument divided by 10 plus an offset of 0.1
		// for example: -10 / 10 + 0.1 = -0.9
		translation.x = float(x) / 10.0f + offset;
		// for-loop argument divided by 10 plus an offset of 0.1
		// for example: -10 / 10 + 0.1 = -0.0
		translation.y = float(y) / 10.0f + offset;
		// for example: translation vec2 = (-0.9, -0.9) 
		// increment index by 1
		translations[index++] = translation;
	}
}
```

Here we create a set of 100 translation vectors that contains an offset vector for all positions in a 10x10 grid. In addition to generating the translation array, we'd also need to transfer the data to the vertex shader's uniform array.

```
shader.use();
for(unsigned int i = 0; i < 100; i++)
{
	shader.setVec2(("offsets[" + s))
}
```
