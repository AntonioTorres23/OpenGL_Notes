Between the vertex and the fragment shader there is an optional shader stage called the geometry shader. A geometry shader takes as input a set of vertices that form a single primitive e.g. a point or a triangle. The geometry shader can then transform these vertices as it sees fit before sending them to the next shader stage. What makes the geometry shader interesting is that it is able to convert the original primitive (set of vertices) to completely different primitives, possibly generating more vertices than were initially given. 

We're going to throw you right into the deep by showing you an example of a geometry shader.

```
#version 330 core
layout (points) in;
layout (line_strip, max_vertices = 2) out;

void main()
{
	gl_Position = gl_in[0].gl_Position + vec4(-0.1, 0.0, 0.0, 0.0);
	EmmitVertex();
	
	gl_Position = gl_in[0].gl_Position + vec4(0.1, 0.0, 0.0, 0.0);
	EmmitVertex();
	
	EndPrimitive;
}
```

Note that the Geometry Shader is a separate shader file that is provided into your shader program. 

This requires additional code in your shader class to process, link, and compile the geometry shader. 

At the start of a geometry shader we need to declare the type of primitive input we're receiving from the vertex shader. We do this by declaring a layout specifier in front of the **in** keyword. This input layout qualifier can take any of the following point values.

- `points`: when drawing `GL_POINTS` primitives (1).
- `lines`: when drawing `GL_LINES` or `GL_LINE_STRIP` (2).
- `lines_adjacency`: `GL_LINES_ADJACENCY` or `GL_LINE_STRIP_ADJACENCY` (4).
- `triangles`: `GL_TRIANGLES`, `GL_TRIANGLE_STRIP` or `GL_TRIANGLE_FAN` (3).
- `triangles_adjacency` : `GL_TRIANGLES_ADJACENCY` or `GL_TRIANGLE_STRIP_ADJACENCY` (6).

These are almost all the rendering primitives we're able to give to rendering calls like `glDrawArrays`. If we'd chosen to draw vertices as `GL_TRIANGLES` we should set the input qualifier to triangles. The number within the parenthesis represents the minimal number of vertices a single primitive contains. 

We also need to specify a primitive type that the geometry shader will output and we do this via a layout specifier in front of the **out** keyword. Like the layout qualifier, the output layout qualifier can take several primitive values.

- `points`
- `line_strip`
- `triangle_strip`

With just these 3 output specifiers we can create almost any shape we want from the input primitives. To generate a single triangle for example we'd specify `triangle_strip` as the output and output 3 vertices. 

The geometry shader also expects us to set a maximum number of vertices it outputs (if you exceed this number, OpenGL won't draw the *extra* vertices) which we can also do within the layout qualifier of the **out** keyword. In this particular case we're going to output a `line_strip` with a maximum, number of 2 vertices. 

In case you're wondering what a line strip is: a line strip binds together a set of two points to form one continuous line between them with a minimum of 2 points. Each extra point results in a new line between the new point and the previous point as you can see in the following image with 5 point vertices. 

![[Pasted image 20251009121813.png]]

To generate meaningful results we need some way to retrieve the output from the previous shader stage. GLSL gives us a **built-in** variable called `gl_in` that internally (probably) looks something like this.

```
in gl_Vertex
{
	vec3 gl_Position;
	float gl_PointSize;
	float gl_ClipDistance[];
} gl_in[];
```

Here it is declared as an **interface block** as discussed from the Advance GLSL notes that contains a few interesting variables of which the most interesting one is `gl_Position` that contains the vector we set as the vertex shader's output. 

Note that it is declared as an array, because most render primitives contain more than 1 vertex. The geometry shader receives **all** vertices of a primitive as its input. 

Using the vertex data from the vertex shader stage, we can generate new data with 2 geometry shader functions called **`EmitVertex`** and **`EndPrimitive`**. The geometry shader expects you to generate/output at lease one of the primitives you specified as output. In our case we want to at least generate on line strip primitive. 

```
#version 330 core
layout (points) in;

layout (line_strip, max_points = 2) out; 

void main()
{
	gl_Position = gl_in[0].gl_Position + vec4(-0.1, 0.0, 0.0, 0.0);
	EmitVertex(); 
	
	gl_Position = gl_in[0].gl_Position + vec4(1.0, 0.0, 0.0, 0.0);
	EmmitVertex();
	
	EndPrimitive();
}
```

Each time we call `EmitVertex`, the vector currently set to `gl_Position` is added to the output primitive. Whenever `EndPrimitive` is called, all emitted vertices for this primitive are combined into the specified output render primitive. By repeatedly calling `EndPrimitive`, after one or more `EndPrimitive` calls, multiple primitives can be generated. This particular case emits two vertices that where translated by a small offset from the original vertex position and then calls `EndPrimitive`, combining the two vertices into a single line strip of 2 vertices.

Now that you (sort of) know how geometry shaders work you can probably guess what this geometry shader does. This geometry shader takes a point primitive as its inputs and creates a horizontal line primitive with the input point at its center. If we were to render this it looks something like this. 

![[Pasted image 20251009161548.png]]

Not very impressive yet, but it's interesting to consider that this output was generated using just the following render call.

`glDrawArrays(GL_POINTS, 0, 4);`

While this is a relatively simple example, it does show you how we use geometry shaders to (dynamically) generate new shapes on the fly. Later in these notes we'll discuss a few interesting effects that we can create using geometry shaders, but for now we're going to start with a simple example. 

**Using Geometry Shaders**

To demonstrate the use of a geometry shader, we're going to render a really simple scene where we draw 4 points on the z-plane in normalized device coordinates. The coordinates of the points are. 

```
float points[] =
{
	-0.5, 0.5, // top-left
	 0.5, 0.5, // top-right
	 0.5, -0.5, // bottom-right
	-0.5, -0.5 // bottom-left 
};
```

The vertex shader needs to draw the points on the z-plane so we'll create a basic vertex shader.

```
#version 330 core
layout (location = 0) vec2 aPos;

void main()
{
	gl_Position = vec4(aPos.x, aPos.y, 0.0, 1.0);
}
```

And we'll output the color green for all points which we code directly in the fragment shader.

```
#version 330 core
out vec4 FragColor;

void main()
{
	FragColor = vec4(0.0, 1.0, 0.0, 1.0);
}

```

Generate a VAO and a VBO for the points' vertex data and then draw them via `glDrawArrays`

```
shader.use();
glBindVertexArray(VAO);
glDrawArrays(GL_POINTS, 0, 4);
```

The result is a dark scene with 4 (difficult to see) green points. 

![[Pasted image 20251010154948.png]]

But didn't we already learn to do all this? Yes, and now we're going to spice this little scene up by adding geometry shader magic to this scene. 

For learning purposes we're first going to create what is called a **pass-through** geometry shader that takes a point primitive as its input and passes it to the next shader unmodified. 

```
#version 330 core
layout (points) in;
layout (points, max_vertices = 2) out;

void main()
{
	gl_Position = gl_in[0].gl_Position;
	EmmitVertex();
	EndPrimitive();
}
```

By now this geometry shader should be fairly easy to understand. It simply emits the unmodified vertex position it received as input and generates a point primitive. 

A geometry shader needs to be compiled and linked to a shader program just like the vertex and fragment shader, but this time we'll create the shader using `GL_GEOMETRY_SHADER` as the shader type. 

```
geometryShader = glCreateShader(GL_GEOMETRY_SHADER);
glShaderSource(geometryShader, 1, &gShaderCode, NULL);
glCompileShader(geometryShader);
[...]
glAttachShader(program, geometryShader);
glLinkProgram(program);
```

The shader compilation code is the same as the vertex and fragment shaders. Be sure to check for compile or linking errors.

If you'd compile and run you should be looking at a result that looks a bit like this. 

![[Pasted image 20251010160427.png]]

It's exactly the same without the geometry shader. It's a bit dull, I'll admit that, but the fact that we were still able to draw the points means that the geometry shader works, so now it's time for the funky stuff. 

**Let's Build Houses**

Drawing points and lines isn't that interesting so we're going to get a little creative by using the geometry shader to draw a house for us at the location of each point. We can accomplish this by setting the output of the geometry shader to `triangle_strip` and draw a total of three triangles. Two for the square house and one for the roof. 

A triangle strip in OpenGL is a more efficient way to draw triangles with fewer vertices. After the first triangle is drawn, each subsequent vertex generates another triangle next to the first triangle: each 3 adjacent vertices will form a triangle. If we have a total of 6 vertices that form a triangle strip we'd get the following triangles. (1,2,3), (2,3,4), (3,4,5), (4,5,6); forming a total of 4 triangles. A triangle strip needs at least 3 vertices and will generate N-2 triangles; with 6 vertices we created 6-2=4 triangles. The following image illustrate this. 

![[Pasted image 20251010161643.png]]

Using a triangle strip as the output of the geometry shader we can easily create the house shape we're after by generating 3 adjacent triangles in the correct order. The following image shows in what order we need to draw what vertices to get the triangles we need with the blue dot being the input point. 

![[Pasted image 20251010162020.png]]

This translates to the following geometry shader. 

```
#version 330 core
layout (points) in;
layout (triangle_strip, max_vertices = 5) out;

void build_house(vec4 position)
{
	gl_Position = position + vec4(-0.2, -0.2, 0.0, 0.0); // 1: bottom-left
	EmitVertex();
	gl_Position = position + vec4(0.2, -0.2, 0.0, 0.0);  // 2: bottom-right
	EmmitVertex();
	gl_Position = position + vec4(-0.2, 0.2, 0.0, 0.0);  // 3: top-left
	EmmitVertex();
	gl_Position = position + vec4(0.2, 0.2, 0.0, 0.0);   // 4: top-right
	EmmitVertex();
	gl_Position = position + vec4(0.0, 0.4, 0.0, 0.0);   // 5: top
	EmmitVertex();
	EndPrimitive();
}

void main()
{
	build_house(gl_in[0].gl_Position);
}
```

This geometry shader generates 5 vertices, with each vertex being being the point's position plus an offset to form one large triangle strip. The resulting primitive is then rasterized and the fragment shader runs the entire triangle strip, resulting in a green house for each point we've rendered. 

![[Pasted image 20251010163338.png]]

You can see that each house indeed consists of 3 triangles - all drawn using a single point in space. The green houses do look a bit boring though, so let's liven it up a bit by giving each house a unique color. To do this we're going to add an extra vertex attribute in the vertex shader with color information per vertex and direct it to the geometry shader that further forwards it to the fragment shader.

The updated vertex data is given below. 

```
float points[] =
{
	-0.5f, 0.5f, 1.0f, 0.0, 0.0, // top-left
	0.5f, 0.5f, 0.0f, 1.0f, 0.0, // top-right
	0.5f, -0.5f, 0.0f, 0.0f, 1.0f, // bottom-right
	-0.5f, -0.5f, 1.0f, 1.0f, 0.0f // bottom-left
};
```

Then we update the vertex shader to forward the color attribute to the geometry shader using an interface block.

```
#version 330 core
layout (location = 0) in vec2 aPos;
layout (loccation = 1) in vec3 aColor;

out VS_OUT
{
	vec3 color;
} vs_out;

void main()
{
	gl_Position = vec4 (aPos.x, aPos.y, 0.0, 1.0);
	vs_out.coclor = aColor;
}

```

Then we also need to declare the same interface block (with a different interface name) in the geometry shader

```
in VS_OUT
{
	vec3 color
} gs_in[];
```

Because the geometry shader acts on a set of vertices as its input, its input data from the vertex shader is always represented as arrays of vertex data even though we only have a single vertex right now. 

We don't necessarily have to use interface blocks to transfer data to the geometry shader. We could have also written it as.

`in vec3 outColor[];`

This works if the vertex shader forwarded the color vector as `out vec3 outColor;`. However, interface blocks are easier to work with in shaders like the geometry shader. In practice, geometry shader inputs can get quite large and grouping them in one large interface block array makes a lot more sense. 

We should also declare an output color vector for the next fragment shader stage. 

`out vec3 fColor;`

Because the fragment shader expects only a single (interpolated) color it doesn't make sense to forward multiple colors. The `fColor` vector is thus not an array, but a single vector. When emitting a vertex, that vertex will store the last stored value in `fColor` as that vertex's output value. For the houses, 