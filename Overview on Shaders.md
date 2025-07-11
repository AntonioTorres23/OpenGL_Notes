
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

