
**GLSL** 

Shaders are written in the C-like language GLSL. GLSL is tailored for use with graphics and contains useful features specifically targeted at vector and matrix manipulation. 

Shaders always begin with a version declaration, followed by a list of input and output variables, uniforms its main function. Each shader's entry point is at its `main` function where we process any input variables and output the results in its output variables. Don't worry if you don't know what uniforms are, we'll get to those shortly. 

A shader typically has the following structure.

```
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

void main()
{
	// process input(s) and do some weird graphics stuff
	...
	// output processed struff to output variable
	out_variable_name = weird_stuff_we_processed;
}
```

When we're talking specifically about the vertex shader each input variable is also known as a **vertex attribute**. There is a maximum number of vertex attributes we're allowed to declare limited by the hardware. OpenGL guarantees there are always at least 16 4-component vertex attributes available, but some hardware may allow for more which you can retrieve by querying `GL_MAX_VERTEX_ATTRIBS`. 

```
int nrAttributes;
glGetIntergerv(GL_MAX_VERTEX_ATRRIB, &nrAttributes);
std::cout << "Maximum nr of vertex attributes supported" << nrAttributes <<                                                                          std::endl;
```

This often returns the minimum of 16 which should be more than enough for most purposes. 

**Types**

GLSL has, like any other programming language, data types for specifying what kind of variable we want to work with. GLSL has most of the default basic types we know from languages like C: `int`, `float`, `double`, `uint`, and `bool`. GLSL also features two container types that we'll be using a lot, namely **vectors** and **matrices**. 

**Vectors**

A vector in GLSL is a 2, 3, or 4 component container for any of the basic types just mentioned. They can take the following form (n represents the number of components).

- `vecn`: the default vector of n floats.
- `bvecn`: a vector of n booleans. 
- `ivecn`: a vector of n integers.
- `uvecn`: a vector of n unsigned integers.
- `dvecn`: a vector of n double components.

Most of the time we will be using the basic `vecn` since floats are sufficient for most of our purposes. 

Components of a vector can be accessed via `vec.x` where x is the first component of the vector. You can use `.x`, `.y`, `.z`, and `.w` to access their first, second, third and fourth component respectively. GLSL also allows you to use `rgba` for colors or `stpq` for texture coordinates, accessing the same components. 

The vector datatype allows for some interesting and flexible component selection called **swizzling**. Swizzling allows us to use syntax like this. 

```
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

You can use any combination of up to 4 letters to create a new vector (of the same type) as long as the original vector has those components; it is not allowed to access the `.z` component of a `vec2` for example. We can also pass vectors as arguments to different vector constructor calls, reducing the number of arguments required.

```
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```

Vectors are thus a flexible datatype that we can use for all kinds of input and output. You will see plenty of examples when learning OpenGL on how we can creatively manage vectors. 

**Ins and Outs**

Shaders are nice little programs on their own, but they are part of a whole and for that reason we want to have inputs and outputs on the individual shaders so that we can move stuff around. GLSL defined the `in` and `out` keywords specifically for that purpose. Each shader can specify inputs and outputs using those keywords and wherever an output variable matches with an input variable of the next shader stage they're passed along. The vertex and fragment shader differ a bit though. 

The vertex shader **should** receive some form of input otherwise it would be pretty ineffective. The vertex shader differs in input, in that it receives its input straight from the vertex data. To define how the vertex data is organized we specify the input variables with location metadata so we can configure the vertex attributes on the CPU.  We've seen this previously with the syntax as `layout (location = 0)`. The vertex shader thus requires an extra layout specification for its inputs so we can link it with the vertex data. 

It is also possible to omit the layout `(location = 0)` specifier and query for the attribute location in your OpenGL code via `glGetAttribLocation`, but I'd prefer to set them in the vertex shader. It is easier to understand and saves you (and OpenGL) some work. 

The other exception is that the fragment requires a vec4 color output variable, since the fragment shaders need to generate a final output color. If you fail to specify an output color in your fragment shader, the color buffer output for those fragments will be undefined (which usually means OpenGL will render them either black or white). 

So if we want to send data from one shader to the other we'd have to declare an output in the sending shader and a similar input in the receiving shader. When the types and names are equal on both sides OpenGL will link those variables together and then it is possible to send data between shaders (this is done when linking a program object). Here is an example of sending and receiving data within GLSL shaders.

**Vertex Shader**

```
#version 330 core
layout (location = 0) in vec3 aPos; // the position variable has attribute position
                                    // 0
out vec4 vertexColor; // specify a color output to the fragment shader

void main()
{
	gl_Position = vec4(aPos, 1.0); // see how we directly give a vec3 to vec4's
								   // constructor
	vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // set the output variable to dark red
											// color
}
```

**Fragment Shader**

```
#version 330 core
out vec4 FragColor;

in vec4 vertexColor; // the input variable from the vertex shader (same name and                        // type)

void main()
{
	FragColor = vertexColor;
}
```

You can see we declared a `vertexColor` variable as a `vec4` output that we set in the vertex shader and we declare a similar `vetexColor` input in the fragment shader. Since they both have the same type and name, the `vertexColor` in the fragment shader is linked to the `vertexColor` in the vertex shader. Because we set the color to a dark-red color in the vertex, the resulting fragments should be dark-red as well. The following image shows the output.

![[Pasted image 20250711164940.png]]

We just managed to send a value from the vertex shader to the fragment shader. Let's spice it up a bit and see if we can send a color from our application to the fragment shader. 

**Uniforms**

**Uniforms** are another way to pass data from our application on the CPU to the shaders on the GPU. Uniforms are however slightly different compared to vertex attributes. First of all, uniforms are **global**. Global, meaning that a uniform variable is unique per shader program object, and can be accessed from any shader at any stage in the shader program. Second, whatever you set the uniform to, uniforms will keep their values until they're reset or updated. 

To declare a uniform in GLSL we simply add the `uniform` keyword to a shader with a type and a name. From that point on we can use the newly declared uniform in the shader. Let's see if this time we can set the color of the triangle via a uniform.

```
#version 330 core
out vec4 FragColor;

uniform vec4 ourColor; // we set this variable in the OpenGL code

void main()
{
	FragColor = ourColor;
}
```

We declared a uniform `vec4` `ourColor` in the fragment shader and set the fragment's output color to the content of this uniform value. Since uniforms are global values, we can define them in any shader stage we'd like so no need to go through the vertex shader again to get something to the fragment shader. We're not using this uniform in the vertex shader so there's no need to define it here. 

If you declare a uniform that isn't used anywhere in your GLSL code the compiler will silently remove the variable from the compiled version which in the cause for several errors. 

The uniform is currently empty; we haven't added any data to the uniform yet so let's try that. We first need to find the index/location of the uniform attribute in our shader. Once we have the index/location of the uniform, we can update its values. Instead of passing a single color to the fragment shader, let's spice things up by changing the color over time.

```
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

First, we retrieve the running time in seconds via `glfwGetTime()`. Then, we vary the color in the range of 0.0-1.0 using the sin function and store the result in `greenValue`.

Then we query for the location of `ourColor` uniform by using `glGetUniformLocation`. We supply the shader program and the name of the uniform (that we want to retrieve the location from) to the query function. If `glGetUniformLocation` returns -1, it could not find the location. Lastly we can set the uniform value using the `glUnifrom4f` function. Note that finding the uniform does not require you to use the shader program first, but updating a uniform does require you to use the program (by calling `glUseProgram`), because it sets the uniform on the currently active shader program. 

Because OpenGL is in its core a C library it does not have native support for function overloading, so wherever a function can be called with different types OpenGL defines new functions for each type required; The function requires a specific postfix for the type of the uniform you want set. A few of the possible postfixes are.

- `f`: the function expects a float as its value.
- `i`: the function expects an integer as its value. 
- `ui`: the function expects an unsigned integer as its value. 
- `3f`: the function expects 3 floats as its value. 
- `fv`: the function expects a float vector/array as its value. 

Whenever you want to configure an option of OpenGL simply pick the overloaded function that corresponds with your type. In our case we want to set 4 floats of the uniform individually so we pass our data via `glUniform4f` (note that we also could've used the `fv` version). 

Now that we know how to set the values of uniform values, we can use them for rendering. If we want the color to gradually change, we want to update this uniform every frame, otherwise the triangle would maintain a single solid color if we only set it once. So we calculate the `greenValue` and update the uniform each render iteration. To simplify that statement we do all of this within the while while loop of our application. 

```
while(!glfwWindowShouldClose(window))
{
	// input
	processInput(window);

	// render
	// clear the colorbuffer
	glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
	glClear(GL_COLOR_BUFFER_BIT);

	// Be sure to activate the shader
	glUseProgram(shaderProgram);

	// update the uniform color
	float timeValue = glfwGetTime();
	float greenValue = sin(timeValue) / 2.0f + 0.5f;
	int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
	glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);

	// now render the triangle
	glBindVertexArray(VAO);
	glDrawArrays(GL_TRAINGLES, 0, 3);

	// swap buffers and poll IO events
	glfwSwapBuffers(window);
	glfwPollEvents();
}
```

The code is a relatively straightforward adaption of the previous code. This time, we update a uniform value each frame before drawing the triangle. If you update the uniform correctly you should see the color of your triangle gradually change from green to black and back to green. 

Check the source code [here](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/3.1.shaders_uniform/shaders_uniform.cpp) if you are stuck. 

As you can see, uniforms are a useful tool for setting attributes that may change every frame, or for interchanging data between your application and your shaders, but what if we want to set a color for each vertex? In that case we'd have to declare as many uniforms as we have vertices. A better solution would be to include more data in the vertex attributes is what which is what we're going to do now. 

**More attributes**

We can fill a VBO, configure vertex attribute pointers and store it all in a VAO. This time, we also want to add color data to the vertex data. We're going to add color data as 3 floats to the vertices array. We assign a red, green, and blue color to each of the corners of our triangle respectively. 

```
float vertices[] =
{
	// positions        // colors
	0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f, // bottom right
   -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f, // bottom left
    0.0f, 0.5f, 0.0f,   0.0f, 0.0f, 1.0f // top 
};
```

Since we now have more data to send to the vertex shader, it is necessary to adjust the vertex shader to also receive our color value as a vertex attribute input. Note that we set the location of the `aColor` attribute to 1 with the layout specifier. 

```
#version 330 core
layout (location = 0) in vec3 aPos; // the position variable has attrib position 0
layout (location = 1) in vec3 aColor; // the color variable has attrib position 1

out vec3 ourColor; // outputs a color to fragment shader

void main()
{
	gl_Position = vec4(aPos, 1.0);
	ourColor = aColor; // set ourColor to the input color we got from vertex data
}
``` 

