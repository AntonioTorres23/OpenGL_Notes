
This part won't really show you super advanced cool features that give an enormous boost to your scene's visual quality. This part goes more or less into some interesting aspects of GLSL and some nice tricks that may help you in your future endeavors. Basically some good to knows and features that may make your life easier when creating OpenGL applications in combinations with GLSL. 

We'll discuss some interesting built-in variables, new ways to organize shader input and output, and a very useful tool called uniform buffer objects. 

**GLSL's Built-in Variables**

Shaders are extremely pipelined, if we need data from any other source outside of the current shader we'll have to pass data around. We learned to do this via vertex attributes, uniforms, and samplers. There are however a few extra variables defined by GLSL prefixed with  `gl_` that gives us extra means to gather and/or write data. We've already seen two of them in the notes so far: `gl_Position` that is the output vector of the vertex shader, and the fragment shader's `gl_FragCoord`. 

We'll discuss a few interesting built-in input and output variables that are built-in in GLSL and explain how they may benefit us. Note that we won't discuss all built-in variables that exist in GLSL so if you want to see all built-in variables you can check OpenGL's [wiki](https://www.khronos.org/opengl/wiki/Built-in_Variable_\(GLSL\)).

**Vertex Shader Variables**

We've already seen `gl_Position` which is the clip-space output position vector of the vertex shader. Setting `gl_Position` in the vertex shader is a strict requirement if you want to render anything on screen. Nothing we haven't seen before. 

**`gl_PointSize`**

One of the render primitives we're able to choose from is `GL_POINTS` in which case each single vertex is a primitive and rendered as a point. It is possible to set the size of the points being rendered via OpenGL's `glPointSize` function, but we can also influence this value in the vertex shader. 

One output variable defined by GLSL is called `gl_PointSize` that is a float variable where you can set the point's width and height by pixels. By setting the point's size in the vertex shader we get per-vertex control over this point's dimensions. 

Influencing the point sizes in the vertex shader is disabled by default, but if you want to enable this you'll have to enable OpenGL's `GL_PROGRAM_POINT_SIZE`. 

`glEnable(GL_PROGRAM_POINT_SIZE);`

A simple example of influencing point size is by setting the point size equal to the clip-space position's z value which is equal to the vertex's distance to the viewer. The point size should then increase the further we are from the vertices as the viewer. 

```
void main()
{
	gl_Position = projection * view * model * vec4(aPos, 1.0);
	gl_PointSize = gl_Position.z;
}
```

The result is that the point's we've drawn are rendered larger the more we move away from them. 

![[Pasted image 20251007161928.png]]

You can imagine that varying the point size per vertex is interesting for techniques like particle generation. 

**`gl_VertexID`**

The `gl_Position` and `gl_PointSize` are output variables since their value is read as output from the vertex shader; we can influence the result by writing to them. The vertex shader also gives us an interesting *input variable*, that we can only read from, called `gl_VertexID`. 

The integer variable `gl_VertexID` holds the current ID of the vertex 