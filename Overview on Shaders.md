
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

You can see we declared a `vertexColor` variable as a `vec4` output that we set in the vertex shader and we declare a similar `vetexColor` input in the fragment shader. Since they both have the same type and name, the `vertexColor` in the fragment shader is linked to the `vertexColor` in the vertex shader. 