
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

Then we take this function `glCheckError_` and create a pre-processor function called `glCheckError` 

In case you're unaware of what the preprocessor directives `__FILE__` and `__LINE__` are: these variables get replaced during compile time with the respective file and line they were compiled in. If we decide to stick with a large number of these `glCheckError` calls in our codebase it's helpful to more precisely know which `glCheckError` call returned the error. 

```
glBindBuffer(GL_VERTEX_ARRAY, vbo);
glCheckError()
```
