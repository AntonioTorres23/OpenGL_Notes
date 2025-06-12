VBO is short for Vertex Buffer Object and is where vertices are stored in a GPUs memory. 

First you create a VBO object using OpenGLs built in library function:

	`unsigned int VBO;`
	`glGenBuffers(1, &VBO);`

You need to create an integer variable called VBO or whatever name to point to the address within the code. 

The one in the OpenGL function above just states how many VBO objects you want to create. In this case just 1. 

Then you bind the VBO variable to a GL_ARRAY_BUFFER using the `glBindBuffer` function:

	`glBindBuffer(GL_ARRAY_BUFFER, VBO);`

This allows your VBO object to be used any time a GL_ARRAY_BUFFER is called.

Next we use the function `glBufferData` to put in whatever vertex data we have in our C++ file and put it into the buffer's memory:

	`glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);`

The parameters in this function mean that we copy our vert data into the object that is currently bound to the GL_ARRAY_BUFFER.

Then, we specify a size of our vert data in bytes, using the built in size of function that C++ comes with by default helps with this.

Then we specify the array we made previously in the program containing the data we want to send to the buffer. 

Lastly is how we want to draw out these data points. GL_STATIC_DRAW is good for what we are doing now since we only use one set of data that is reused many times. However, there are different methods of drawing verts so it depends on what you are doing. 

Now when creating our shader we have made a separate header file that helps clean up the main C++ file so it will look different than the first few lessons on LearnOpenGL's website. But, the same idea is applied to getting shaders and applying them to your OpenGL project. 

Fist we need to create or provide a vertex and fragment shader program. This can be an external file or a C-string variable pointer. If it's external, we need to create a program or function that reads our files and then stores the data in a variable. This is done via the shader_S.h file we made along side the LearnOpenGL lesson. 

However, within that header file is the same thing. 

Create a shader:

	`vertexShader = glCreateShader(GL_VERTEX_SHADER);` 
for vertex or:

	`fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);`
for fragment.

Remember that a vertex shader deals with the coordinates for the thing you want to draw while the fragment shader deals with the general output of the shape(s). Either it be color or other factors. 

The next function applies for both shaders, 

`glShaderSource(YOUR_SHADER_OBJECT_NAME_HERE, 1, &(your shader source code here), NULL);`

What this function's parameters does is first take the previous shader object that we created. 

Then the second parameter takes how many strings of GLSL code we want to include with this shader source. In this case we are only taking one.

Thirdly, we include our GLSL source code that was either in a c-string variable or external file that is being extracted with the shader_s.h file. 

Lastly, 4th specifies an array of string lengths, since we only have one string, we will leave it as NULL. 

After this, for both vert and frag shaders you compile your shader using an OpenGL function:

	`glCompileShader(YOUR_SHADER_OBJECT_NAME_GOES_HERE);`


Within our shader_s.h header file we have error checking for anything going wrong and providing logs for us. 

But here is the raw source code for information:

`int success;`

`char infoLog[512];`

`glGetShaderiv(YOUR_SHADER_OBJECT_NAME_GOES_HERE, GL_COMPILE_STATUS, &success);`

These 2 variables and 1 OpenGL function are the basis for our error checking with compiling our two shaders. 

The success variable is there as a placeholder to see if the compile failed or worked. 

The info log array gives us a way to store our info so we can read what is going on. 

The built in OpenGL function is a way for us to check if compilation was successful. With our shader object going first, what status we want to get (in this case the compile status), and our int specifying if it was successful or not (I'm assuming it goes off of 1 if true 0 if false like bit-wise operators). 

So with these variables you can craft a if statement such as this:

	`if(!success)
	{
			glGetShaderInfoLog(YOUR_SHADER_OBJECT_NAME_HERE, 512, NULL, infoLog); 
			std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog               << std::endl;
	}`


This function within the if statement gets info from the shader object, takes a maximum amount of bytes that we made with the infoLog array which was 512.  The length of the string that was returned which we don't need so its set to null. Lastly, the infoLog array which is where the information is stored. 

After this we send it to default output which allows us to read it. 


Lastly, we need a shader program to link the vert and frag shader together. Here is an example of the code here:

	`unsigned int shaderProgram;
	 shaderProgram = glCreateProgram();`

Similar to creating the vert and shader objects we make a variable then use a function to assign that variable as an object. 

Now we need to attach the vert and frag shaders to the shader program using `glAttachShader`:

		`glAttachShader(shaderProgram, vertexShader); 
		glAttachShader(shaderProgram, fragmentShader);

Lastly, we link the programs together using `glLinkProgram`:
			 
		glLinkProgram(shaderProgram);`

Similar to the previous error checking of the vert and shader objects, we can also do this for the program linking:

	`glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
	 if(!success) 
	 { 
		 glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog); 
		 ... 
	 }`

The function in the if statement works exactly like the `glGetShaderInfoLog` one. Just with different info and status. 

Now you can use your new shader program with this function:

	`glUseProgram(shaderProgram);`


The two previous shader objects should be deleted due to their use not being needed after they are linked to the shader program. 

Now its time to learn about Linking Vertex Attributes. 

This process allows us to specify any input we want in the form of vertex attributes. 

Vertex buffer data is stored in 32bit or 4byte floating point values.

Each position is composed of 3 of those values.

There is no space between each set of 3 values. 

The first value in the data is at the beginning of the buffer. 

We accomplish this by using the function `glVertexAttribPointer`:

	`glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);`

The parameters inside this function are: 
1. which vertex attribute we want to configure. This applies to our vertex shader where we specified our layout (location = 0).
2. Specifies the size of the vertex attribute which is 3 from the vertex shader being called vec3 and containing 3 values (x, y, z).  3.
3. Data type being used which is GL_FLOAT.
4.  This has to do with normalizing the data which changes the value of numbers depending on the value and if its signed or not. No need to worry about this parameter so just set it to GL_FALSE.
5. This is the stride and how much space we have between vertex attributes. This changes I've noticed with how much data you are working with and how tightly packed you want your arrays so tread with caution. Although it should be bigger than the size of the data type you're working with which is float. 
6. This one is weird but, then void* just means you can take any data type. I believe this was how the parameter works within the OpenGL Lib Code so don't worry too much about it. But after that is the offset of where the position begins in the data buffer. Since this position data is at the start of the array we will set it to zero. But with more attribute pointers this will need to change overtime. 

After this, we use `glEnableVertexAttribArray(0);` 

All this does is enable a position within an index of the function at which you want to work with. 

So if you had another attribute pointer for something else, you would set it to 1, 2, and so on depending on how many you have. 


VAOs are required when working with vertex inputs, however they provide a lot of help when drawing an object multiple times. 

VAO or vertex array object can be bound like a VBO and any vertex attribute calls from there will be stored within the VAO. 

Pretty much the same as creating a VBO: 

`unsigned int VAO; 
`glGenVertexArrays(1, &VAO);`

You create a variable with the name you want and then specify how many VAOs you want to create. In this case just one. After this, you provide the address of your variable. 

Now your object is created. 

After this you need to bind your vertex array with this function: 

	`glBindVertexArray(VAO);`

This allows the VAO to store the vertex attribute config that we made prior and which VBO to use. 

Typically you want to bind and configure all the VAOs. To draw an object, we take our corresponding VAO, bind it, and then draw the object and unbind the VAO again.

Use the function `glUseProgram()` with your shader program object to activate it. 

Then, bind your vertex array as stated previously. 

Then draw your triangles with `glDrawArrays(GL_TRIANGLES, 0, 3);`

So it should look like this within your source code:


`glUseProgram(shaderProgram); 
`glBindVertexArray(VAO); 
`glDrawArrays(GL_TRIANGLES, 0, 3);`

Then hopefully, your object should appear.

The parameters in `glDrawArrays()` are as follows: 

1.  Take in some type of primitive to render, in this case, GL_TRIANGLES
2. This tells OpenGl what vertex array to start on which we started at 0.
3. This argument asks us how many vertices we would like to draw. 


EBO or Element Array Objects allow us to preform indexed drawing. This type of drawing lets us specify what vertices we would like to draw. 

This example is typically with a square in which we use two triangles to draw it. 

To start, use an array of vertices like previously that make up a square using two triangles. Then create a second array of your indices stating which order you want your triangles drawn. 

Here is an example of the two from the website:

`float vertices[] = { 0.5f, 0.5f, 0.0f, // top right 
				    `0.5f, -0.5f, 0.0f, // bottom right
				     `-0.5f, -0.5f, 0.0f, // bottom left 
				     `-0.5f, 0.5f, 0.0f // top left };`
`unsigned int indices[] = { // note that we start from 0! 
						`0, 1, 3, // first triangle 
						`1, 2, 3 // second triangle };`

So as we can see, you have 4 coordinates here that make up a square if you use a graphing calculator. Next in the indices array you have 2 triangles that are specifying what order they want those coordinates to be drawn in with each section being x, y, and z.  

Creating a Element Buffer Object is similar to the VAO and VBO, you create a variable for the EBO, then use `glGenBuffers()` and state how many objects you want to make and an address to the variable you made previously.

`unsigned int EBO; 
`glGenBuffers(1, &EBO);`

Similar to our VBO we want to bind the EBO and copy the indices into the buffer with `glBindBuffer` and `glBufferData()`. Only this time, we are using the GL_ELEMENT_ARRAY_BUFFER type. 

Another example from the code:


`glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO); 
`glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);`


After this, you can draw your elements with `glDrawElements()` which is similar to `glDrawArrays()` only instead of arrays you are drawing elements:

`glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);`


Now for some reason, LearnOpenGL does not specify the order of which these objects should be created or bound until the end of the lesson so use this example from the website as a general reference since you need the VAO to be bound last so the EBO will also be stored within there to store the binding.

It should be structured like this when binding and getting your VBO, VAO, and EBO set up for drawing. 

`// ..:: Initialization code :: .. 
`// 1. bind Vertex Array Object 
`glBindVertexArray(VAO); 
`// 2. copy our vertices array in a vertex buffer for OpenGL to use glBindBuffer(GL_ARRAY_BUFFER, VBO); 
`glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); 
`// 3. copy our index array in a element buffer for OpenGL to use glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO); 
`glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW); 
`// 4. then set the vertex attributes pointers 
`glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0); 
`glEnableVertexAttribArray(0); 
`[...] 
`// ..:: Drawing code (in render loop) :: .. 
`glUseProgram(shaderProgram); 
`glBindVertexArray(VAO); 
`glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0); 
`glBindVertexArray(0);`

EXTRA NOTE:

If you want to enable wireframe mode, use this:

`glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);`









