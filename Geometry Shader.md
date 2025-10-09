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

Each time we call `EmitVertex`, the vector currently set to `gl_Position` is added to the output primitive. Whenever `EndPrimitive` is called, all emitted vertices for this primitive are combined into the specified output render primitive. By repeatedly calling `EndPrimitive`