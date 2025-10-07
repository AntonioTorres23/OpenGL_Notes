
This part won't really show you super advanced cool features that give an enormous boost to your scene's visual quality. This part goes more or less into some interesting aspects of GLSL and some nice tricks that may help you in your future endeavors. Basically some good to knows and features that may make your life easier when creating OpenGL applications in combinations with GLSL. 

We'll discuss some interesting built-in variables, new ways to organize shader input and output, and a very useful tool called uniform buffer objects. 

**GLSL's Built-in Variables**

Shaders are extremely pipelined, if we need data from any other source outside of the current shader we'll have to pass data around. We learned to do this via vertex attributes, uniforms, and samplers. There are however a few extra variables defined by GLSL prefixed with  `gl_` that gives us extra means to gather and/or write data. We've already seen two of them in the notes so far: `gl_Position` that is the output vector of the vertex shader, and the fragment shader's `gl_FragCoord`. 

We'll discuss a few interesting built-in input and output variables that are built-in in GLSL and explain how they may benefit us. Note that we won't discuss all built-in variables 