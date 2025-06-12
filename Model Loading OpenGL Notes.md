
In this note we will grab the meshes from the model and put it into our C++ code using assimp. The goal is to create another class that represents a model in its entirety, that is, a model that contains multiple meshes as well as textures. 

Here is the model class:

`class Model`
`{
	`public:`
		`Model(char *path)`
		`{`
			`loadModel(path);`
		`}`
		`void Draw(Shader &shader);`
	`private:`
		`// model data`
		`vector<Mesh> meshes;`
		`string directory;`

		`void loadModel(string path);`
		`void processNode(aiNode *node, const aiScene *scene);`
		`Mesh processMesh(aiMesh *mesh, const aiScene *scene);`
		`vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type,            string typeName);`
`};`

The model class contains a vector of Mesh objects and requires us to give it a file location in its constructor. It then loads the file away via the loadModel function that is called in the constructor. The private functions are all designed to process a part of assimp's important routine and we'll cover them shortly. We also store the directory of the file path that we'll later need when loading textures. 

The Draw function loops over each of the meshes to call their respective Draw function from the meshes class. 

`void Draw(Shader &shader)`
`{`
	`for(unsigned int i = 0; i < meshes.size(); i++)`
		`meshes[i].Draw(shader)`
`}`

Importing a 3D model into OpenGL

To import a model and translate it to our own structure, we first need to include the appropriate headers of Assimp:

`#include <assimp/Importer.hpp>`
`#include <assimp/scene.h>`
`#include <assimp/postprocess.h>`

The first function we're calling is loadModel, that's directly called from the constructor. Within loadModel, we use Assimp to load the model into a data structure of Assimp called a scene object. Which is the root object of Assimp's data interface. Once we have a scene object, we can access all the data we need from the loaded model. 

The great thing about assimp is that it neatly abstracts from all the technical details of loading all the different file formats and does this with a single one-liner. 

`Assimp::Importer importer;`
`const aiScene *scene = importer.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs);`

We first declare an Importer object from Assimp's namespace and then call its ReadFile function. The function expects a file path and several post-processing options as its second argument. 
Assimp allows us to specify several options that forces Assimp to do extra calculations/operations on the imported data. By setting aiProcess_Triangulate we tell Assimp that if the model does not (entirely) consist of triangles, it should transform all primitive shapes to triangles first. The aiProcess_FlipUVs flips the texture coordinates on the y-axis where necessary during processing.   '

 Here is the complete load model function:

`void loadModel(string path)`
`{`
	`Assimp::Importer importer;`
	`const aiScene *scene = importer.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs);`

	`if(!scene || scene->mFlags & AI_SCENE_FLAGS_INCOMPLETE || !scene->mRootNode)`
	`{`
		`cout << "ERROR::ASSIMP::" << import.GetErrorString() << endl;`
		`return;` 
	`}`
	`directory = path.substr(0, path.find_last_of('/'));`

	`processNode(scene->mRootNode, scene);`
`}`

After we load the model, we check if the scene and the root node of the scene are not null and check one of its flags to see if the returned data is incomplete. If any of these error conditions are met, we report the error retrieved from the importers GetErrorString function and return. We also retrieve the directory path of the given file path.

If nothing went wrong, we want to process all of the scene's nodes. We pass the first node to the recursive processNode function. Because each node contains a set of children we want to first processes the node in question, and then continue processing all the node's children and so on. This fits a recursive structure, so we'll be defining a recursive function. A recursive function is a function that does some processing and recursively calls the same function with different parameters until a certain condition is met. In our case the exit condition is met when all the nodes have been processed. 

As you may remember from Assimp's structure, each node contains a set of mesh indices where each index points to a specific mesh located in the scene object. We thus want to retrieve these mesh indices, retrieve each mesh, process each mesh, and then do this all again for each of the node's children of nodes. The content of processNode function is shown below:

`void processNode(aiNode *node, const aiScene *scene)`
`{`
	`// process all the node's meshes if any`
	`for(unsigned int i = 0; i < node->mNumMeshes; i++)`
	`{`
		`aiMesh *mesh = scene->mMeshes[node->mMeshes[i]];`
		`meshes.push_back(processMesh(mesh, scene));`
	`}`
	`// do the same for each of its children`
	`for(unsigned int i = 0; i < node->mNumChildren; i++)`
	`{`
		`processNode(node->mNumChildren[i], scene);`
	`}`
`}`

We first check each of the node's mesh indices and retrieve the corresponding mesh by indexing the scene's mMeshes array. The returned mesh is then passed to the processMesh function that returns a Mesh object that we can store in the meshes list/vector.

Once all the meshes have been processed, we iterate through all the node's children and call the same processNode function for each child. One a node no longer has any children, the recursion stops. 

You may think why not just use a while or for loop to complete this process. We use a recursive function is because the initial idea for using nodes like this is that it defines a parent-child relation between meshes. An example for this would be importing a car object. When you translate a car mesh you want to translate the car and all its children (engine mesh, tires mesh, etc) translate as well. Such a system is easily created using parent-child relations.

Right now however we are not using such a system, but it is generally recommended to stick with this approach for whenever you want extra control over your mesh data. These node-like relations are after all defined by artists who create the models. 

Assimp to Mesh

Translating an aiMesh object to a mesh object of our own is not too difficult. All we need to do, is access each of the mesh's relevant properties and store them in our own object. The general structure of the processMesh function then becomes:

`Mesh processMesh(aiMesh *mesh, const aiScene *scene)`
`{`
	`vector<Vertex> vertices;`
	`vector<unsigned int;> indices;`
	`vector<Texture> textures;`

	`for(unsigned int i = 0; i < mesh->mNumVertices, i++)`
	`{`
		`Vertex vertex;`
		`// process vertex positions, normals, and texcoords`
		`[...]`

		`vertices.push_back(vertex);`
	`}`

	`// process indices`
	`[...]`

	`// process material`
	`if(mesh->mMaterialIndex >= 0);`
	`{`
		`[...]`
	`}`

	`return Mesh(vertices, indices, textures);`
		
`}`

Processing a mesh is a 3-part process: retrieve all the vertex data, retrieve the mesh's indices, and finally retrieve the relevant material data. The processed data is stored in one of the 3 vectors and from those a mesh is created and returned to the functions caller.

Retrieving the vertex data is pretty simple. We define a Vertex structure that we add to the vertices array after each loop iteration. We loop for as many vertices there exist within the mesh (retrieved via mesh->mNumVertices). Within the iteration we want to fill this structure with all the relevant data. For vertex positions, this is done as follows.

`glm::vec3 vector;`
`vector.x = mesh->mVertices[i].x;`
`vector.y = mesh->mVertices[i].y;`
`vector.z = mesh->mVertices[i].z;`
`vertex.Position = vector;`

Note that we define a temporary vec3 for transferring Assimp's data to. This is necessary as Assimp maintains its own data types for vector, matrices, strings, etc. and they don't convert that well to glm data types. 

Assimp calls their vertex array mVertices. 

The procedure for normals is the same:

`vector.x = mesh->mNormals[i].x;`
`vector.y = mesh->mNormals[i].y;`
`vector.z = mesh->mNormals[i].z`
`vertex.Normal = vector;`

Texture coordinates are the similar, but Assimp allows a model to have up to 8 different texture coordinates per vertex. we only care about the first set of texture coordinates. We'll also want to check if the mesh actually has texture coordinates.

`if(mesh->mTextureCoords[0])`
`{`
	`glm::vec2 vec;`
	`vec.x = mesh->mTextureCoords[0][i].x;`
	`vec.y = mesh->mTextureCoords[0][i].y;`
	`vertex.TexCoords = vec;`
`}`
`else`
	`vertex.TexCoords = glm::vec2(0.0f, 0.0f);`

The vertex structure is now filled with the required vertex attributes and we can push it to the back of the vertices vector at the end of the iteration. 

Indices

Assimp's interface defines each mesh as having an array of faces. Where each face represents a single primitive, which in our case is aiProcess_Triangulate. A face contains the indices of the vertex we need to draw in what order for its primitive. So if we iterate over all the faces and store all the face's indices in the indices vector. We are all set:

`for(unsinged int i = 0; i < mesh->mNumFaces; i++)`
`{`
	`aiFace face = mesh->mFaces[i]`
	`for(unsigned int j = 0; j < face.mNumIndices; j++)`
	`{`
		`indices.push_back(face.mIndices[j]);`
	`}`
`}`

While the outer loop has finished, we now have a complete set of vertices and index data for drawing the mesh via glDrawElements.

However, we also want to process a mesh's material as well.

Material

Similar to nodes, a mesh only contains an index to a material object. To retrieve the material of a mesh, we need to index the scene's mMaterials array. The mesh's material index is set in its mMaterialIndex property, which we can also query to check if the mesh contains a material or not. 

`if(mesh->mMaterialIndex >=0)`
`{`
	`aiMaterial *material = scene->mMaterials[mesh->MaterialIndex];`

	`vector<Texture> diffuseMaps = loadMaterialTextures(material,                        aiTextureType_DIFFUSE, "texture_diffuse");`

	`textures.insert(textures.end(), diffuseMaps.begin(), diffuseMaps.End());`
	`vector<Texture> specularMaps = loadMaterialTextures(material,                       aiTextureType_SPECULAR, "texture_specular");`

	`textures.insert(textures.end(), specularMaps.begin(), specularMaps.end());`

`}`

We first retrieve the aiMaterial object from the scene's mMaterials array. Then we want to load the mesh's diffuse and/or specular textures. A material object internally stores an array of texture locations for each texture type. 

The different texture types are all prefixed with aiTextureType_. We use a helper function called loadMaterialTextures to retrieve, load, and initialize the textures from the material. The function returns a vector of Texture structures that we store at the end of the model's textures vector. 

The loadMaterialTextures function iterates over all the texture locations of the given texture type, retrieves the texture's file location and then loads and generates the texture and stores the information in a Vertex structure. It looks like this:

`vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type, string typeName)`
`{`
	`vector <Texture> textures;`
	`for(unsigned int i = 0; i < mat->GetTextureCount(type); i++)`
	`{`
		`aiString str;`
		`mat->GetTexture(type, i, &str);`
		`Texture texture;`
		`texture.id = TextureFromFile(str.C_Str(), directory);`
		`texture.type = typeName;`
		`texture.path = str;`
		`textures.pushback(texture);`
	`}`
	`return textures;`
`}`

We first check the amount of textures stored in the material via its GetTextureCount function that expects one of the texture types we've given. We retrieve each of the texture's file locations via the GetTexture function that stores the result in an aiString. We then use another helper function called TextureFromFile that loads a texture (with stb_image.h) for us and returns the texture's ID. You can check the complete code listing at the end for its content if you're not sure how such as function is written. 

Note that we make the assumption that texture file paths in model files are local to the actual model object in the same directory. We can then simply concatenate the texture location string and the directory string we retrieved earlier in the loadModel function to get the complete texture path. That's why the GetTexture function also needs the directory string. 

Some models found over the internet use absolute paths for their texture locations, which won't work on each machine. In that case you probably want to edit the file to use local paths for the textures if possible. 

An Optimization

We're not completely done yet, since there is still a large (but not completely necessary) optimization we want to make. Most scenes re-use several of their textures onto several meshes. Think of a house again that has a granite texture for its walls. This texture could also be applied to the floor, ceiling, staircase, perhaps a table, and maybe even a small well close by. Loading textures is not a cheap operation and in our current implementation a new texture is loaded and generated for each mesh, even though the exact same texture could have been loaded several times before. This quickly bottlenecks your modal loading. So we're going to add one small tweak to the model code by storing all of the loaded textures globally. Whenever we want to load a texture, we first check if it hasn't been loaded already. If so, we take that texture and skip the entire loading routine, saving us a lot of processing power. To be able to compare textures we need to store their path as well:

`struct Texture {`
	`unsigned int id;`
	`string type;`
	`string path; // we store the path of the texture to compare with others`
`};`

Then we store all the loaded textures in another vector declared at the top of the model's class file as a private variable.

`vector<Texture> textures_loaded;`

In the loadedMaterialTextures function, we want to compare the texture path with all the textures_loaded vector to see if the current texture path equals any of those. If so, we skip the texture loading/generation part and simply use the located texture struct as the mesh's texture. The updated function is shown below:

`vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type, string typeName)`
`{`
	`vector<Texture> textures;`
	`for(unsigned int i = 0; i < mat->GetTextureCount(type); i++)
	`{
		`aiString str;`
		`mat->GetTexture(type, i, &str);`
		`bool skip = false;`
		`for(unsigned int j = 0; j < textures_loaded.size, j++);`
		`{`
			`if(std::strcmp(textures_loaded[j].path.data(), str.C_Str()) == 0)`
			`{`
				`textures.push_back(textures_loaded[j]);`
				`skip = true;`
				`break;`
			`}`
		`}`
	`}`
	`if(!skip)`
	`{`
		`// if texture hasn't been loaded already, load it`
		`Texture texture;`
		`texture.id = TextureFromFile(str.C_str(), directory);`
		`texture.type = typeName;`
		`texture.path = str.C_Str();`
		`textures.push_back(texture);`
		`textures_loaded.pushback(texture) // add to loaded textures-`
	`}`
	`return textures;`
`}`








``




