
Graphics programming can be a lot of fun, but it can also be a large source of frustration whenever something isn't rendering just right, or perhaps not even rendering at all! Seeing as most of what we do involves manipulating pixels, it can be difficult to figure out the cause of error whenever something doesn't work the way it's supposed to. Debugging these kinds of *visual* errors is different than what you're used to when debugging errors on the CPU. We have no console to output text to, no breakpoints to set on GLSL code, and no way of easily checking the state of GPU execution. 

In these notes we'll look into several techniques and tricks of debugging your OpenGL program. Debugging in OpenGL is not too difficult to do and getting a grasp of its techniques definitely pays out in the long run. 

**`glGetError()`**

The moment you incorrectly use OpenGL (like configuring a buffer without binding any) it will take notice and generate one or more user error flags behind the scenes. We can query these error flags using a function named `glGetError` that checks the error flag(s) set and returns an error value if OpenGL got misused. 

`GLenum glGetError();`

The moment `glGetError` is called, it returns either an error flag or no error at all. The error codes that `glGetError` can return are listed below. 

| Flag                               | Code | Description                                                                       |
| ---------------------------------- | ---- | --------------------------------------------------------------------------------- |
| `GL_NO_ERROR`                      | 0    | No user error reported since the last call to `glGetError`.                       |
| `GL_INVALID_ENUM`                  | 1280 | Set when an enumeration parameter is not legal.                                   |
| `GL_INVALID_VALUE`                 | 1281 | Set when a value parameter is not legal.                                          |
| `GL_INVALID_OPERATION`             | 1282 | Set when the state for a command is not legal for its given parameters.           |
| `GL_STACK_OVERFLOW`                | 1283 | Set when a stack pushing operation causes a stack overflow.                       |
| `GL_STACK_UNDERFLOW`               | 1284 | Set when a stack popping operation occurs while the stack is at its lowest point. |
| `GL_OUT_OF_MEMORY`                 | 1285 | Set when a memory allocation operation cannot allocate (enough) memory.           |
| `GL_INVALID_FRAMEBUFFER_OPERATION` | 1286 | Set when reading or writing to a framebuffer that is not complete.                |

Within OpenGL's function documentation you can always find the error codes a function generates the moment it is incorrectly used. For instance, if you take a look at the documentation of [`glBindTexture`](http://docs.gl/gl3/glBindTextur%65) function, you can find all the user error codes it could generate under the *Errors* section.

The moment an error flag is set, no other error flags will be reported. Furthermore, the moment `glGetError` is called it clears all error flags (or only one if on a distributed system, see note below). This means that if you call `glGetError` once at the end of each frame and it returns an error, you can't conclude this was the only error, and the source of the error could've been anywhere in the frame. 

Note that when OpenGL runs distributed like frequently found on X11 systems, other user error codes can still be generated as long as they have different error codes. Calling `glGetError` then only resets one of the error code flags instead of all of them. Because of this, it is recommended to call `glGetError` inside a loop.


```
glBindTexture(GL_TEXTURE_2D, tex);
std::cout << glGetError() << std::endl; // RETURNS 0 (no error)

glTexImage2D(GL_TEXTURE_3D, 0, GL_RGB, 512, 512, 0, GL_RGB, GL_UNSIGNED_BYTE, data); // RETURNS 1280 (invalid enum)

glGenTextures(-5, &textures); // RETURNS 1281 (invalid value)

std::cout << glGetError() << std::endl; // RETURNS 0 (no error)
```

The great thing about `glGetError` is that it makes it relatively easy to pinpoint where any error may be and to validate the proper use of OpenGL. Let's say you get a black screen and you have no idea what's causing it: is the framebuffer not properly set? Did I forget to bind a texture? By calling `glGetError` all over your codebase, you can quickly catch the first place an OpenGL error starts showing up.

By default `glGetError` only prints error numbers, which isn't easy to understand unless you've memorized the error codes. It often makes sense to write a small helper function to easily print out the error strings with where the error check function was called. 

```
GLenum glCheckError_(const char *file, int line)
{
	GLenum errorCode;
	while ((errorCode = glGetError()) != GL_NO_ERROR)
	{
		std::string error;
		switch(errorCode)
		{
			case GL_INVALID_ENUM:      error = "INVALID_ENUM"; break;
			case GL_INVALID_VALUE:     error = "INVALID_VALUE"; break;
			case GL_INVALID_OPERATION: error = "INVALID_OPERATION"; break;
			case GL_STACK_OVERFLOW:    error = "STACK_OVERFLOW"; break;
			case GL_STACK_UNDERFLOW:   error = "STACK_UNDERFLOW"; break;
			case GL_OUT_OF_MEMORY:     error = "OUT_OF_MEMORY"; break;
			case GL_INVALID_FRAMEBUFFER_OPERATION: error =                                     "INVALID_FRAMEBUFFER_OPERATION"; break;
		}
		std::cout << error << "|" << file << " (" << line << ")" << std::endl;
	}
	return errorCode;
}
#define glCheckError() glCheckError_(__FILE__, __LINE__)
```

To explain this in my own words, we define the function `glCheckError_` within normal C++ source code. The function in question takes a `glEnum` variable called `errorCode`. A while loop when `errorCode` is not equal to the enumeration value `GL_NO_ERROR` takes place, it will go through a switch case statement and grab the correct error flag that matches. `errorCode` in the while loop Boolean statement is set to the OpenGL built-in function `glGetError()`. Another string variable called `error` will then be set with the related matching switch case statement providing a summary of what the error flag grabbed indicates. After it finds a match and sets the `error` string variable with a brief summary, it then breaks out of the while loop and sends default output to the console in which the `error` string, file location, and line where the error occurred are displayed. As well as the `errorCode` `GLenum` variable is returned to end the function. 

Then we take this function `glCheckError_` and create a pre-processor function called `glCheckError` which just reuses the C++ source code function, but uses the `__FILE__` and `__LINE__` as its "arguments" or directives.  What these two directives do is whenever the function `glCheckError()` gets called, `__FILE__` during compile time will be replaced with the file location and `__LINE__` will be replaced with the respective line number.   
 
In case you're unaware of what the preprocessor directives `__FILE__` and `__LINE__` are: these variables get replaced during compile time with the respective file and line they were compiled in. If we decide to stick with a large number of these `glCheckError` calls in our codebase it's helpful to more precisely know which `glCheckError` call returned the error. 

```
glBindBuffer(GL_VERTEX_ARRAY, vbo);
glCheckError()
```

This will give us the following output.

![[Pasted image 20251219150013.png]]

`glGetError` doesn't help you too much as the information it returns is rather simple, but it does often help you catch typos or quickly pinpoint where in your code things went wrong; a simple but effective tool in your debugging toolkit. 

**Debug Output**

When it comes to GLSL, we unfortunately don't have access to a function like `glGetError` nor the ability to step through the shader code. When you end up with a black screen or completely wrong visuals, it's often difficult to figure out if something's wrong with the shader code. Yes, we have the compilation error reports that report syntax errors, but catching semantic errors is another beast. 

One frequently used trick to figure out what is wrong with a shader is to evaluate all the relevant variables in a shader program by sending them directly to the fragment shader's output channel. By outputting shader variables directly to the output color channels, we can convey interesting information by inspecting the visual results. For instance, let's say we want to check if a model has correct normal vector. We can pass them (either transformed or untransformed) from the vertex shader to the fragment shader where we'd then output the normals as follows. 

```
#version 330 core
out vec4 FragColor;
in vec3 Normal;
[...]

void main()
{
	[...]
	FragColor.rgb = Normal;
	FragColor.a = 1.0f;
}
```

By outputting a (non-color) variable to the output color channel like this we can quickly inspect if the variable is, as far as you can tell, displaying the correct values. If, for instance, the visual result is completely black it is clear the normal vectors aren't correctly passed to the shaders; and when they are displayed it's relatively easy to check if they're (sort of) correct or not. 

![[Pasted image 20251219153647.png]]

From the visual results we can see the world-space normal vectors appear to be correct as the right sides of the backpack model is mostly colored red (which would mean the normals roughly point (correctly) towards the positive x axis). Similarly, the front side of the backpack is mostly colored toward the positive z axis (blue). 

This approach can easily extend to any type of variable you'd like to test. Whenever you get stuck and suspect there's something wrong with your shaders, try displaying multiple variables and/or intermediate results to see at which part of the algorithm something's missing or seemingly incorrect. 

**OpenGL GLSL Reference Compiler**

Each driver has its own quirks and tidbits; for instance, NVIDIA drivers are more flexible and tend to overlook some restrictions on the specification, while ATI/AMD drivers tend to better enforce the OpenGL specification (which is the better approach in my opinion). The results of this is that shaders on one machine may not work on the other due to driver differences. 

With years of experience you'll eventually get to learn the minor difference between GPU vendors, but if you want to be sure your source code runs on all kinds of machines you can directly check your shader code against the official specification using OpenGL's GLSL [reference compiler](https://www.khronos.org/opengles/sdk/tools/Reference-Compiler/). You can download the so called **GLSL** lang validator binaries from [here](https://www.khronos.org/opengles/sdk/tools/Reference-Compiler/) or its complete source code from [here](https://github.com/KhronosGroup/glslang).

Given the binary GLSL lang validator you can easily check your shader code by passing it as the binary's first argument. Keep in mind that the GLSL lang validator determines the type of shader by a list of fixed extensions.

- `.vert`: vertex shader
- `.frag`: fragment shader
- `.geom`: geometry shader
- `.tesc`: tessellation control shader
- `.tese`: tessellation evaluation shader
- `.comp`: compute shader

Running the GLSL reference compile shader is as simple as.

`glsllangvalidator shaderFile.vert`

Note that if it detects no error, it returns no output. Testing the GLSL reference compiler on a broken vertex shader gives the following output. 

![[Pasted image 20251222103903.png]]

It won't show you the subtle differences between AMD, NVidia, or Intel GLSL compilers, nor will it help you completely bug proof your shaders, but it does as least help you check your shaders against the direct GLSL specification. 

**Framebuffer Output**

Another useful trick for your debugging toolkit is displaying a framebuffer's content(s) in some pre-defined region of your screen. You're likely to use [framebuffers](https://learnopengl.com/Advanced-OpenGL/Framebuffers) quite often and, as most of their magic happens behind the scenes, it's sometimes difficult to figure out what's going on. Displaying the content(s) of a framebuffer on your screen is a useful trick to quickly see if things look correct. 

Note that displaying the contents (attachments) of a framebuffer as explained here only works on texture attachments, not render buffer objects. 

Using a simple shader that only displays a texture, we can easily write a small helper function to quickly display any texture at the top right of the screen.

```
// vertex shader
#version 330 core
layout (location = 0) in vec2 position; 
layout (location = 1) in vec2 texCoords; 

out vec2 TexCoords;

void main()
{
	gl_Position = vec4(position, 0.0f, 1.0f);
	TexCoords = texCoords; 
}

// fragment shader
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;

uniform sampler2D fboAttachment;

void main()
{
	FragColor = texture(fboAttachment, TexCoords);
}
```

```
void DisplayFramebufferTexture(unigned int textureID)
{
	if (!notInitalized)
	{
		// initalize shader and vao w/ NDC vertex coordinates at top-right of scrn
	}
	glActivateTexture(GL_TEXTURE0);
	glUseProgram(shaderDisplayFBOOutput);
		glBindTexture(GL_TEXTURE_2D, textureID);
		glBindVertexArray(vaoDebugTexturedRect);
			glDrawArrays(GL_TRIANGLES, 0, 6);
		glBindVertexArray(0);
	glUseProgram(0);
}

int main()
{
	[...]
	while (!glfwWindowShouldClose(window))
	{
		[...]
		DisplayFramebufferTexture(fboAttachment0);
		
		glfwSwapBuffers(windows);
	}
	
	
}
```


This will give you a nice little window at the corners of your screen for debugging framebuffer output. Useful, for example, for determining if the normal vectors of the geometry pass in a deferred renderer look correct. 

![[Pasted image 20251222122919.png]]

You can of course extend such a utility function to support rendering more than one texture. This is a quick and dirty way to get continuous feedback from whatever is in your framebuffer(s). 

**External Debugging Software**

When all else fails there is still the option to use a 3rd party tool to help us in our debugging efforts. Third party applications often inject themselves in the OpenGL drivers and are able to intercept all kinds of OpenGL calls to give you a large array of interesting data. These tools can help you in all kinds of ways like: profiling OpenGL function usage, finding bottlenecks, inspecting buffer memory, and displaying textures and framebuffer attachments. When you're working on (large) production code, these kinds of tools can become invaluable in your development process. 

I've listed some of the more popular debugging tools here; try out several of them to seer which fits your needs the best. 

**RenderDoc** 

RenderDoc is a great (completely [open source](https://github.com/baldurk/renderdoc)) standalone debugging tool. To start a capture, you specify the executable you'd like to capture and a working directory. The application then runs as usual, and whenever you want to inspect a particular frame, you let RenderDoc capture one or more frames at the executable's current state. Within the captured frame(s) you can view the pipeline state, all OpenGL commands, buffer storage, and textures in use.

![[Pasted image 20251222124059.png]]

**CodeXL**

[CodeXL](https://gpuopen.com/compute-product/codexl/) is a GPU debugging tool released as both a standalone tool and a Visual Studio plugin. CodeXL gives a good set of information and is great for profiling graphics applications. CodeXL also works on NVidia or Intel cards, but without support for OpenCL debugging. 

![[Pasted image 20251222130546.png]]

I personally don't have much experience with CodeXL since I found RenderDoc easier to use, but I've included it anyways as it looks to be a pretty solid tool and developed by one of the larger GPU manufacturers. 

**NVIDIA Nsight**

NVIDIA's popular [Nsight](https://developer.nvidia.com/nvidia-nsight-visual-studio-edition) GPU debugging tool is not a standalone tool, but a plugin to either the Visual Studio IDE or the Eclipse IDE (NVIDIA now has a [standalone version](https://developer.nvidia.com/nsight-graphics) as well).