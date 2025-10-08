
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

The integer variable `gl_VertexID` holds the current ID of the vertex we're drawing. When doing *indexed rendering* (with `glDrawElements`) this variable holds the current index of the vertex we're drawing. When drawing without indices (`glDrawArrays`) this variable holds the number of the currently processed vertex since the start of the start of the render call.

**Fragment Shader Variables**

Within the fragment shader we also have access to some interesting variables. GLSL gives us two interesting input variables called `gl_FragCoord` and `gl_FrontFacing`. 

**`gl_FragCoord`**

We've seen the `gl_FragCoord` a couple of times before and during the discussion of depth testing, because then z component of the `gl_FragCoord` vector is equal to the depth value of that particular fragment. However, we can also use the x and y component of that vector for some interesting effects. 

The `gl_FragCoord`'s x and y component are the window- or screen-space coordinates of the fragment, originating from the bottom-left of the window. We specified a render window of 1200x1000 with `glViewport` so the screen-space coordinates of the fragment will have x values between 0-1200, and y values between 0-1000. 

Using the fragment shader we could calculate a different color based on the screen coordinate of the fragment. A common usage for the `gl_FragCoord` variable is for comparing visual output of different fragment calculations, as usually seen in tech demos. We could for example split the screen in two by rendering one output to the left side of the window and another output on the right side of the window. An example fragment shader that outputs a different color based on the fragment's screen coordinates is given below. 

```
void main()
{
	if(gl_FragCoord.x < 600)
		FragColor = vec4(1.0, 0.0, 0.0, 1.0);
	else
		FragColor = vec4(0.0, 1.0, 0.0, 1.0);
}
```

Because the width of the window is equal to 1200, whenever a pixel's x-coordinate is less than 800 it must be at the left side of the window and we'll give that fragment a different color. 

![[Pasted image 20251008115533.png]]

We can now calculate two completely different fragment shader results and display each of them on a different side of the window. This is great for testing out different lighting techniques for example. 

**`gl_FrontFacing`**

Another interesting input variable in the fragment shader is the `gl_FrontFacing` variable. In the Face Culling section we mentioned that OpenGL is able to figure out if a face is a front or back face due to the winding order of the vertices. The `gl_FrontFacing` variable tells us if the current fragment is part of a front-facing or back-facing face. We could, for example, decide to output different colors for all back faces. 

The `gl_FrontFacing` variable is a bool that is true if the fragment is part of a front face and false otherwise. We could create this cube this way with a different texture on the inside than on the outside. 

```
#version 330 core
out vec4 FragColor; 

in vec2 TexCoords;

uniform sampler2D frontTexture;
uniform sampler2D backTexture;

void main()
{
	if(gl_FrontFacing)
		FragColor = texture(frontTexture, TexCoords);
	else
		FragColor = texture(backTexture, TexCoords);
}
```

If we take a peek inside the container we can now see a different texture being used.

![[Pasted image 20251008120838.png]]

Note that if you enabled face culling you won't be able to see any faces inside the container and using `gl_FrontFacing` would be pointless. 

**`gl_FragDepth`**

The input variable `gl_FragCoord` is an input variable that allows us to read screen-space coordinates and get the depth value of the current fragment, but it is a read-only variable. We can't influence the screen-space coordinates of the fragments, but it is possible to set the depth-value of the fragment within the shader. 

To set the depth value in the shader we write any value between 0.0 and 1.0 to the output variable. 

`gl_FragDepth = 0.0; // this fragment now has a depth value of 0.0`

If the shader does not write anything to `gl_FragDepth`, the variable will automatically takes its value from `gl_FragCoord.z`. 

Setting the depth value manually has a major disadvantage however. That is because OpenGL disables early depth testing as soon as we write to the `gl_FragDepth` in the fragment shader. It is disabled, because OpenGL cannot know what depth value the fragment will have before we run the fragment shader, since the fragment shader may change this value. 

By writing to `gl_FragDepth` you should take this performance penalty into consideration. From OpenGL 4.2 however, we can still sort of mediate between both sides by redeclaring `gl_FragDepth` variable at the top of the fragment shader with a depth condition.

`layout (layout (depth_<condition>) out float gl_FragDepth);`

This condition can take the following values:

- any: The default value. Early depth testing is disabled.
- greater: You can only make the depth value larger compared to `gl_FragCoord.z`
- less: You can only make the depth value smaller compared to 
