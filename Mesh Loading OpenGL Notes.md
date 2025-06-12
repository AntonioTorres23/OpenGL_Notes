
1st off, remember to include your assimp lib in the linker input as well as the assimp dll file along side the directory where your main script resides. This had me stuck for like a week and I had to do research to figure out why this is occurring. 

Assimp is a way to load .obj files into OpenGL. It is a popular extended library for loading models. Assimp stands for Open Asset Import Library. 

Thinking back to previous ways we have created 3d models with things such as vectors, indices, and many other things. Doing that all in OpenGL is not easy or realistically capable. This is typically why at most game dev companies, models are first made in 3D software like blender, maya, and many other applications and then, imported into the game engine. 

These 3d Modeling tools are then exported into a file format such as .obj or .dae (COLLADA) file format. This contains all the data needed for a model to be loaded into the engine. However depending on the file type that you export a model to. It can have more or less data. For example .obj is a much older file type and contains the basics such as vertices, colors, diffuse/specular maps. While the .dae file type can have extensive lighting, materials, animation data, cameras, scene info, and many other things. 

The 3d Modeling tools also allow for textures to be applied to them by a process called uv-mapping. 

Assimp's data structure stays the same, regardless of the file format, it abstracts us from all the different file formats out there. 

Assimp loads the entire model object into a scene that contains all the data of the imported model. Assimp then has a collection of nodes where each node contains indices to data stored in the scene object where each node can have any number of children. A simple model of Assimp structure is shown in the picture below.


![[Pasted image 20250417122539.png]]
To further explain the diagram above:
- All the data of the scene/model is contained in the Scene object like all the materials and meshes. It also contains a reference to the root node of the scene.
- The root node of the scene may contain children nodes (like all other nodes) and could have a set of indices that point to mesh data in the scene object's mMeshes array. The scene's mMeshes array contains the actual Mesh objects, the values in the mMeshes array of a node are only indices for the scene's meshes array. 
- A Mesh object itself contains all the relevant data required for rendering, think of vertex positions, normal vectors, texture coordinates, faces, and the material of the object.
- A mesh contains several faces. A face represents a render primitive of the object (triangles, squares, and points). A face contains the indices of the vertices that form a primitive. Because the vertices and the indices are separated, this makes it easy for us to render via an index buffer.
- Finally, a mesh also links to a Material object that hosts several functions to retrieve the material properties of an object. Think of colors and or texture maps (like diffuse or specular maps).


The process goes like this: first load an object into a Scene object, recursively retrieve the corresponding Mesh objects from each of the nodes (we recursively search each node's children), and process each Mesh object to retrieve the vertex data, indices, and its material properties. The result is then a collection of mesh data that we want to contain in a single model object. 

We will be making a Model and Mesh class that load and store imported models using the structure we've just described. We don't render the model as a whole, but we render all the individual meshes that the model is composed of. 

Mesh Class:

We use the Mesh class to transform Assimp's data structures into a formatting that OpenGL understands. A mesh represents a single drawable entity. So let's start by creating a mesh class of our own. 

A mesh should at least have vertices, where each vertex contains a position vector, a normal vector, and a texture coordinate vector. A mesh should also have indices for indexed drawing, and material data in the form of textures (diffuse or specular maps).

Now that we have defined what a mesh should contain, we can define a vertex in OpenGL.

`struct Vertex
`{`
	`glm::vec3 Position;
	`glm::vec3 Normal;
	`glm::vec2 TexCoords;
`};`

We store each of the required vertex attributes in a structure called Vertex. Next to a Vertex struct we also want to organize the texture data in a Texture struct.

`struct Texture
`{
		`unsigned int id;
		`string type;
};`

We store the id of the texture and its type e.g. a diffuse or specular texture. 

Knowing the actual representation of a vertex we can start defining the structure of the mesh class:

`class Mesh`
`{
		`public:`
			`// mesh data`
			 `vector<Vertex> verticies;`
			 `vector<unsigned int> indices;`
			 `vector<Texture> textures;`

			`Mesh(vector<Vertex> vertices, vector<unsigned int> indices,                         vector<Texture> textures);`

			`void Draw(Shader &shader);`
		`private:`
			`// render data`
			`unsigned int VAO, VBO, EBO;`

			`void setupMesh();`
				
`};

The constructor we give the mesh all the necessary data, we initialize the buffers in the setupMesh function, and finally draw the mesh via the Draw function.

Note that we give a shader to the Draw function. By passing the shader to the mesh we can set several uniforms before drawing (like linking samplers to texture units).

The function content of the constructor is pretty straightforward. We simply set the class's public variables with the constructor's corresponding argument variables. We also call the setupMesh function in the constructor.

`Mesh(vector<Vertex> vectices, vector<unsigned int> indices, vector<Texture> textures)`
`{
		`this->vertices = vertices;`
		`this->indices;`
		`this->textures = textures;`

		`setupMesh();`
`}`


Initialization

Thanks to the constructor, we now have large lists of mesh data that we can use for rendering. We do need to setup the appropriate buffers and specify the vertex shader layout via vertex attribute pointers. 

`void setupMesh()`
`{
	`glGenVertexArrays(1, &VAO);`
	`glGenBuffers(1, &VBO);`
	`glGenBuffers(1, &EBO);`

	`glBindVertexArray(VAO);`
	`glBindBuffer(GL_ARRAY_BUFFER, VBO);`

	`glBufferData(GL_ARRAY_BUFFER, verticies.size() * sizeof(Vertex), &vertices[0],      GL_STATIC_DRAW);`

	`glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);`
	`glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size * sizeof(unsigned int),          &indices[0], GL_STATIC_DRAW);`

	`// vertex positions`
	`glEnableVertexAttribArray(0);`
	`glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)0);`
	`// vertex normals`
	`glEnableVertexAttribArray(1);`
	`glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex),                     (void*)offsetof(Vertex, Normal));`
	`// vertex texture coords`
	`glEnableVertexAttribArray(2);`
	`glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, sizeof(Vertex),                     (void*)offsetof(Vertex, TexCoords));`

	`glBindVertexArray(0);`
`}`

The code is not much different from what you'd expect, but a few tricks were used with the help of the Vertex structure.


Structures have a great property in C++ that their memory layout is sequential. That is, if we were to represent a struct as an array of data, it would only contain the structure's variables in sequential order which directly translates to a float (actually a byte) array that we want for an array buffer. For example, if we have a filled Vertex structure, its memory layout would be equal to:

`Vertex vertex;`
`vertex.Position = glm::vec3(0.2f, 0.4f, 0.6f);`
`vertex.Normal = glm::vec3(0.0f, 1.0f, 0.0f);`
`vertex.TexCoords = glm::vec2(1.0f, 0.0f);`
`// = [0.2f, 0.4f, 0.6f, 0.0f, 1.0f, 0.0f, 1.0f, 0.0f];`

What this essentially means is that when the data structure is created and all of these variable are stored within there which is vertex. Every vertex point within the vertex structure goes top to bottom left to right in the order they where created. So the first 3 values in position are first in left to right order. Then, the normal vertices are included left to right, and so on. 

Sequential: "performed or used in sequence."

Thankfully due to this property of data structures, we can directly pass a pointer (&) to a large list of Vertex structs as the buffers data and they translate perfectly to what glBufferData expects as its argument. 

`glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(Vertex), ***&vertices[0]***, GL_STATIC_DRAW);`

Naturally the sizeof operator can also be used on the struct for the appropriate size in bytes. 

Another great use of structs is a preprocessor directive called offsetof  that takes its first argument a structure and as its second argument a variable name of the structure. The macro returns a byte offset that variable from the start of the structure. This is perfect for defining the offset parameter of the glVertexAttribPointer function.

`glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), ***(void*)offsetof(Vertex, Normal))***;`

This offset is now defined using the offsetof macro that, in this case, sets the byte offset of the normal vector equal to the byte offset of the normal attribute in the struct which is 3 floats and thus 12 bytes. Basically, meaning that there are 3 points from the start of the vertex structure the the beginning of the normal vertex. So offsetof does the math from there. 3 points are before the normal variable. Each float point contains 4 bytes. Multiply the amount of data points with float by the amount of bytes it takes per data point (3x4) and you get 12 bytes. Which is our offset value. 

Using a structure not only gets us more readable code, it allows us to extend the structure. If we want another vertex attribute we can simply add it to the structure due to its flexible nature, the rendering code won't break. 

Rendering 

The last function we need to define for the Mesh class to be complete is its Draw function. Before rendering the mesh, we first want to bind the appropriate textures before calling glDrawElements. However, this is somewhat difficult since we don't know from the start how many (if any) textures the mesh has and what type they may have. So how do we set the texture units and samplers in the shaders?

To solve this issue we're going to assume a certain naming convention: each diffuse texture is named texture_diffuseN, and each specular texture should be named texture_specularN where N is any number ranging from 1 to the maximum number of texture samplers allowed. Let's say we have 3 diffuse textures and 2 specular textures for a particular mesh, their texture samplers should then be called:

`uniform sampler2D texture_diffuse1; 
`uniform sampler2D texture_diffuse2; 
`uniform sampler2D texture_diffuse3; 
`uniform sampler2D texture_specular1; 
`uniform sampler2D texture_specular2;`

By this convention we can define as many texture samplers as we want in then shaders (up to OpenGL (2,147,483,647)) and if a mesh actually does contain (so many) textures, we know what their names are going to be. By this convention we can process any amount of textures on a single mesh and the shader developer is free to use as many of those as it wants by defining the proper samplers. 

The code for the Draw function now becomes:

`void Draw(Shader &shader)`
`{
	`unsinged int diffuseNr = 1;`
	`unsigned int specularNr = 1;`
	`for(unsigned int i = 0; i < textures.size(); i++);` 
	`{`
		`glActivateTexture(GL_TEXTURE0 + i);` activate proper texture unit before binding
		`// retrieve texture number (the N in diffuse_textureN)`
		`string number;
		`string name = textures[i].type;
		`if(name == "texture_diffuse")`
			`number = std::to_string(diffuseNr++);`
		`else if(name == "texture_specular")`
			`number = std::to_string(specularNr++);`
		`shader.setInt(("material." + name + number).c_str(), i);`
		`glBindTexture(GL_TEXTURE_2D, textures[i].id);`
	`}`
	`glActivateTexture(GL_TEXTURE0);`
	`// draw mesh`
	`glBindVertexArray(VAO);`
	`glDrawElements(GL_TRIANGLES, indices.size(), GL_UNSIGNED_INT, 0);`
	`glBindVertexArray(0);`
`}`

We first calculate the N-component per texture type and concatenate it to the texture's type string to get the appropriate uniform name. We then locate the appropriate sampler, give it the location value to correspond with the currently active texture unit, and bind the texture. This is also the reason we need the shader in the Draw function. 

We also added "material." to the resulting uniform name because we usually store the textures in a material struct (this may differ per implementation).

Note that we increment the diffuse and specular counters the moment we convert them to a string. In C++ the increment call: var++ returns the variable as is and then increments the variable while ++variable first increments the variable and then returns it. In our case the value passed to std::string is the original counter value. After that the value is incremented for the next round. 

Keep in mind this part is just an abstract concept of developing a mesh class. The full class source code provided on the Learn OpenGL website differs. 




