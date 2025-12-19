
Graphics programming can be a lot of fun, but it can also be a large source of frustration whenever something isn't rendering just right, or perhaps not even rendering at all! Seeing as most of what we do involves manipulating pixels, it can be difficult to figure out the cause of error whenever something doesn't work the way it's supposed to. Debugging these kinds of *visual* errors is different than what you're used to when debugging errors on the CPU. We have no console to output text to, no breakpoints to set on GLSL code, and no way of easily checking the state of GPU execution. 

In these notes we'll look into several techniques and tricks of debugging your OpenGL program. Debugging in OpenGL is not too difficult to do and getting a grasp of its techniques definitely pays out in the long run. 

**`glGetError()`**

The moment you incorrectly use OpenGL (like configuring a buffer without binding any) it will take notice and generate 