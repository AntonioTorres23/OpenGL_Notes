
**Colors**

Previously, we have messed around with colors such as with the Hello Triangle or changing the background color, texture color, etc. Here we will discuss what colors are and start building upon the concept. 

IRL, colors take any known color value with each object having its own colors. In the digital world we need to map the real colors to digital values and therefore not all real-world colors can be represented digitally. Colors are digitally represented using RGB (Red, Green, and Blue). Using a combination of those values, within the range of 0, 1, you can create almost any color. 

Here is an example of creating a coral color within a vector using RGB values:

`glm::vec3 coral(1.0f, 0.5f, 0.31f);`

The color of an object IRL is not the color it actually has, but is the color **reflected** from then object. The colors that aren't absorbed (rejected) by the object is the color we perceive of it. 

We see the sun as a white light which is the sum of many combined colors. If you see something like a blue truck, it would absorb all the white color's sub-colors except the blue color. Since the truck does not absorb the blue color, it is reflected. 

This is the same way lighting works in OpenGL, with defining a light source with a color. If we multiply the color vector by the lighting vector, we get the resulting color vector.

`glm::vec3 lightColor(1.0f, 1.0f, 1.0f);`
`glm::vec3 toyColor(1.0f, 0.5f, 0.31f);`
`glm::vec3 result = lightColor * toyColor; // = (1.0f, 0.5f, 0.31f);`

The toyColor vector absorbs a lot of the white light. As well as reflects various RGB values based on its own color value. Now what if we set that lights value to something like green for example?

`glm::vec3 lightColor(0.0f, 1.0f, 0.0f);`
`glm::vec3 toyColor(1.0f, 0.5f, 0.31f);`
`glm::vec3 result = lightColor * toyColor; // = (0.0f, 0.5f, 0.0f)`

In this version of result, the toyColor has no red or blue light to reflect. The toy also absorbs half of the light's green value. So the toy will be perceived as a dark green color.

You can mess around with all the light vector's RGB values to get some different effects. 

**Creating a Lighting Scene**

We will be using these concepts by applying them to a 3d environment using visual objects.

The first thing we need is an object to cast the light on.

You next want to create a light VAO specifically for the light source. This is due to rendering a completely different object (typically a cube but doesn't have to be). You could also do some transformations on your current object and scale it down and move it away to create the new light source. You do this all in the model matrix within the main C++ file. 


Now that we have both the object and light source, we can multiply the light source's color and object color to get the result like we discussed previously. We do this with first defining two uniform vectors within the global space of the GLSL frag file. Next, within the main function you store the product of your light and object color vectors in your out 4 coordinate vector. 

Here's an example:

`#version 330 core`
`out vec4 FragColor;`

`uniform vec3 objectColor;`
`uniform vec3 lightColor;`

`void main()`
`{`
	`FragColor = vec4(lightColor * objectColor, 1.0);`
`}`

The fragment shader accepts both an object color and a light color from a uniform variable.

Then, within your main C++ file, we take the names of those variables and use our lighting shader header file to specify their color. However first we need to activate our shader before adding our data. 

Here is another example of what the code should look like.

`// Don't forget to activate your shader program first to set the uniform`
`lightingShader.use();`
`lightingShader.setVec3("objectColor", 1.0f, 0.5f, 0,31f);`
`lightingShader.setVec3("lightColor", 1.0f, 1.0f, 1.0f);`

One thing to note is that we will start to update these lighting shaders in the future. The light source will be affected which is something we don't want. We want the light source to stay separated from the object colors and to stay consistent. 

So to achieve this we will create a second set of shaders that we will use to draw the light source. This ensures that nothing will change the light source's color or appearance. 

The light source's vertex shader code is the exact same as the object one so you can just copy and paste that into this one as well. In the light source's fragment shader you just declare an out 4 vector variable for the output color. Then in the main function just set the output variable to store all 4 vectors as 1.0; 

Here is the example source code:

`#version 330 core`
`out vec4 FragColor;`

`void main()`
`{`
	`FragColor = vec4(1.0f); // set all 4 vector values to 1.0`
`}`

When we want to render, we want to render the container object or many other objects using the lighting shader we just defined. When we want to draw the light source we use the light source shaders. 

The main purpose the light source object is to represent where the light is coming from. 

So we need to define a position now where the light object should be. 

`glm::vec3 lightPos(1.2f, 1.0f, 2.0f);`

We then translate the light source cube to the position we declared above. In world space coordinates wise. 

As well as make the cube smaller through scaling it with GLM.

`model = glm::mat4(1.0f);`
`model = glm::translate(model, lightPos);`
`model = glm::scale(model, glm::vec3(0.2f));`

The render code should look something similar to this. 

`lightCubeShader.use();`
`// set the model, view and projection uniforms`
`[...]`
`//draw the light cube object`
`glBindVertexArray(lightcubeVAO);`
`glDrawArrays(GL_TRIANGLES, 0, 36);`

To be fair you'd just draw another object with the new transformations or draw your new object if you are using something like a model. 

**Basic Lighting**

So now we know the basics of changing an objects color based on lighting, we can now start to apply more real world effects on them. 

Phong model lighting is a way graphics developers can simplify the complexity of real-world lighting. 

The major building blocks of the Phong model lighting consists of 3 components: ambient, diffuse, and specular lighting. 

Ambient lighting: even when it is dark there is usually still some light somewhere so objects are never completely dark. To simulate this we use an ambient lighting constant that always gives the object some color.  

![[Pasted image 20250507161413.png]]


Diffuse lighting: simulates the directional impact a light object has on an object. This is the most visually significant component of the lighting model. The more a part of of an object faces the light source, the brighter it becomes. Think of when people teach values in art and how you start off with basic shapes (like a cube). You declare a light source and then start working from there, wherever the light is, you leave that without it being shaded in. Then in the lights that doesn't face the light you make it darker. 

Specular lighting: simulates the bright spot of a light that appears on shiny objects. Specular highlights are more inclined to the color of the light than the color of the light than the color of the object. 

**Ambient Lighting**

Light usually does not come from a single light source, but from many light sources all around us. One of the properties of light is that it can scatter and bounce in many directions, reaching spots that aren't immediately visible. This in the Phong model is called ambient lighting. 

To do this in OpenGL we just declare a constant color that we add to the final resulting color of the object's fragments, thus making it look like there is always some scattered light even when there's not a direct light source. 

Adding ambient lighting to the scene is really easy. We take the light's color, and multiply it with a small constant ambient factor, multiply this with the object's color, and use that as the fragment's color in the cube object's shader.

Here is an example:

`void main()`
`{`
	`float ambientStrength * lightColor;`
	`vec3 ambient = ambientStrength * lightColor;`

	`vec3 result = ambient * objectColor;`
	`FragColor = vec4(result, 1.0);`
`}`

Now if you add this into your fragment shader and run the program, the object should look dark but not black as if there is no color or texture because of the ambient lighting. 

![[Pasted image 20250507161612.png]]

**Diffuse Lighting**

Ambient lighting by itself doesn't the most interesting results, but diffuse lighting however will start to give a significant visual impact on the object. Diffuse lighting gives the object more brightness the closer the light source fragments are aligned to the light rays from a light source. To give you a better understanding of diffuse lighting, take a look at this picture. 

![[Pasted image 20250508110517.png]]

To the left we see the light source with a ray pointing to a single fragment of our object.

We need to measure at what angle the light ray touches the fragment. If the light ray is perpendicular to the objects surface the light has the greatest impact. To measure the angle between the light ray and the fragment we use a normal vector. This is a vector perpendicular to the fragment's surface (this is the yellow arrow in the picture). 

The angle between the Normal vector and the Light Ray vector can then be calculated with a dot product.

Dot product is just multiplying the two vectors of the same position (x * x) (y * y) (z * z). Then adding those products together into a sum (x * x) + (y * y) + (z * z) = dot product. 

The lower the angle between two vectors, the more the dot product is inclined towards the value of 1. When the angle between both vectors is 90 degrees, the dot product becomes 0. The same applies to 0: the larger 0 becomes, the less of an impact the light should have on the fragment's color. 

Note that to get only the cosine of the angle between both vectors we will work with unit vectors (vectors of length 1) so we need to make sure all vectors are normalized. Otherwise, the dot product returns more than just the cosine.

A normalized vector is just a vector with a magnitude (or length ) of exactly one. 

The magnitude of a vector is its length, representing the strength or size of the vector. 

Here is the formula to find the magnitude of a 2d vector: 

|v| = √(x² + y²)

Here is the formula to find the magnitude for 3d vectors:


|v| = √(x² + y² + z²)

Here is how magnitude is used in our context:

- [
    **Computer graphics:**
    
    Magnitude is used in computer graphics to determine the length or size of vectors used to represent objects and transformations.

The resulting dot product thus returns a scalar that we use to calculate the light's impact on the fragment's color, resulting in differently lit fragments based on their orientation towards the light.

So to summarize, what we need to calculate diffuse lighting:

- Normal Vector: vector perpendicular to the vertex surface
- The Directional Light Ray: a direction vector that is the difference vector between the light's position and the fragment's position. To calculate this light ray we need the light's position vector and the fragment's position vector. 

**Normal Vectors**

A normal vector is a (unit) vector that is perpendicular to the surface of a vertex. Since a vertex by itself has no surface (just a single point in space). We retrieve a normal vector by using its surrounding vertices to figure out the surface of a vertex. We can use a trick to calculate the normal vectors for all the object's vertices by using the cross product. 

Cross product: 

Determines a vector perpendicular to two given vectors, often used for finding triangle normals. 

**`a =(ax, ay, az)`**
**`b =(bx, by, bz)`**

`a x b = (ay * bz - az * by, az * bx - ax * bz, ax * by - ay * bx)`

So you are just taking 2 products one of them original and then the other one just with the two points flipped (so if the first one is ay * bz, then you would get the difference from az * by ) then subtracting them and that is your coordinate. 

Also remember the right hand rule applies to OpenGL which means your x coordinate points right, your y coordinate points up, and your z coordinate points front. Think of it like the camera section of learning OpenGL.  Where z allows you to move forward and backwards, x lets you move left and right, and y lets you move up. 


In cases like a cube though you can manually add them into to the vertex data. Try to visualize that the normals are vectors perpendicular to each plane's surface. 

Since we added extra data to the vertex array, we need to update the objects vertex shader:

`#version 330 core`
`layout (location = 0) in vec3 aPos;`
`layout (location = 1) in vec3 aNormal;`
`...`

Note that we added a normal vector to each of the vertices and updated the vertex shader, we should update the the vertex attribute pointer as well. Note that the light source's object uses the same vertex array for its data, but the lamp shader has no use of the newly added vectors. We don't have to update the lamp's shaders or attribute configurations. But we have to at least modify the vertex attribute pointers to reflect the new vertex array's size:

`glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);`
`glEnableVertexAttribArray(0);`

We want to only use the first 3 floats of each vertex and ignore the last 3 floats so we only need to update the stride parameter (spacing of the arrays) to six times the size of the float data type.

It may look inefficient using vertex data that is not completely used by the light shader. But the vertex data is already stored in the GPU's memory from the container object so we don't have to store new data into the GPU's memory. This actually makes it more efficient compared to allocating a new VBO specifically for the light object. 

Or in other words, you are using the already established data within the previous VBO object that was originally used to draw the object. Since it always stays in that buffer data stored into the GPU until the program is killed. 

All of the lighting calculations are done in the fragment shader so we need to froward the normal vectors from the vertex shader to the fragment shader. 

`out vecc3 Normal;`

`void main()`
`{`
	`gl_Position = projection * view * model * vec4(aPos, 1.0);`
	`Normal = aNormal;`
`}`

What's left to do is declare the corresponding input variable in the fragment shader:

`in vec3 Normal;`

**Calculating the Diffuse Color**

We now have the normal vector for each vertex, but we still need the light's position vector and the fragment's position vector. Since the light's position is a static variable, we can declare it as a uniform in the fragment's shader. 

`uniform vec3 lightPos;`

Ant then update the uniform in the main C++ file render loop (or outside of that loop and just in this file because it doesn't change per frame). We use the lightPos vector we declared earlier. 

`lightingShader.setVec3("lightPos", lightPos);

Then the last thing we need is the actual fragment's position. We're going to do all the lighting calculations in the world space so we want a vertex position that is in world space first. We can accomplish this by multiplying the vertex position  attribute with the model matrix only (not the view or projection matrix).

Meaning you are getting the actual position of the object itself, nothing else. Then you are just multiplying it by the coordinates within a vec3 coordinates. 

`out vec3 FragPos; //The position of the object in world space that goes to the Frag`
`out vec3 Normal;`

`void main()`
`{`
	`gl_Position = projection * view * model * vec4(aPos, 1.0);`
	`// Here is where we get the world position of the object;`
	`FragPos = vec3(model * vec4(aPos, 1.0));`
	`Normal = aNormal;`
`}`

Then in the fragment shader, we take in the FragPos variable. 

`in vec3 FragPos;`

This in variable will be interpolated (insert something different in nature into something else) to form the FragPos vector that is the per-fragment world position. Now that all the required variables are set we can start the lighting calculations. 

The first thing to calculate is the direction vector between the light source and the object positions. From the previous section, we know that the light's direction vector is the difference vector light's position vector and the fragment's position vector. So light position - fragment/object position. 

We also want make sure all the relevant vectors end up as unit vectors so we will normalize the normal vectors and the directional/distance vector.

`vec3 norm = normalize(Normal);`
`vec3 lightDir = normalize(lightPos - FragPos);`

When calculating lighting we usually do not care about the magnitude of a vector or their position; we only care about their direction. Because we only care about their direction almost all the calculations are done with unit vectors since it simplifies most calculations (like the dot product). So when doing lighting calculations, make sure you always normalize the relevant vectors to ensure they're actual unit vectors. Forgetting to normalize a vector is a common lighting mistake. 

Next we need to calculate the diffuse impact of the light on the current fragment by taking the dot product between norm and lightDir vectors. The resulting value is then multiplied with the light's color to get the diffuse component, resulting in a darker diffuse component the greater the angle between both vectors.

`float diff = max(dot(norm, lightDir), 0.0);`

`vec3 diffuse = diff * lightColor;`

If the angle between both vectors is greater than 90 degrees, then the result of the dot product will actually become negative and we end up with a negative diffuse component. For that reason we use the max function that returns the highest of both its parameters to make sure the diffuse component never become negative. 

To explain this better, the max built-in function compares two numbers, the dot product of norm and lightDir, or 0.0. So if the dot product result is larger, the diff var will have that dot product. If the dot product is negative, the diff var will be 0. So the diff var will never go under zero. 

Now that we have both an ambient and a diffuse component, we add both colors to each other and then multiply that sum by the color of the object to get the final output color. 

`vec3 result = (ambient + diffuse) * objectColor;`
`FragColor = vec4(result, 1.0);`

The successful output on screen should look like this.

![[Pasted image 20250513143354.png]]

You can see that with the diffuse lighting the cube looks more realistic. 

**One Last Thing**

In the previous section, we passed the normal vector directly from the vector space to the fragment shader. However, the calculations in the fragment shader are all done in world space, so shouldn't we transform the normal vectors to world space coordinates as well? Basically yes, but it's not as simple as multiplying it with a model  matrix.

First of all, normal vectors are only direction vectors and do not represent a specific position in space. Second, normal vectors do not have a homogeneous coordinate (the w component of a vertex position).

**Homogeneous Coordinates**are a system of coordinates used in computer graphics and geometry to simplify and unify mathematical operations like translation, scaling, rotation, and perspective projection.

To showcase the idea in a math way lets look at 2d and 3d homogenous coordinates:

**In 2D**, a point (x , y) in Cartesian coordinates becomes (x,y,1) in homogeneous coordinates.

In **3D**, a point (x, y, z) becomes (x,y,z,1).

In general, a point (x ,y) becomes (wx ,wy ,w) for any non-zero w. When w≠1, you can convert back to Cartesian coordinates by dividing by w:

(x, y) =(xh/w​​, yh/w​​) 

This means that translations should not have any effect on the normal vectors. So if we want to multiply the normal vectors with a model matrix we want to remove the translation part of the matrix, by taking the upper-left 3x3 matrix of the model matrix (note that we could also set the w component of a normal vector to 0 and multiply with the 4x4 matrix).

Second, if the model matrix would preform a non-uniform scale, the vertices would be changed in such a way that the normal vector is not perpendicular to the surface anymore. The following diagram shows the effect such a model matrix (with non-uniform scaling) has on a normal vector. 

![[Pasted image 20250513152253.png]]

Whenever we apply a non-uniform scale (a uniform scale only changes the normals magnitude, not its direction, which can be fixed by normalizing it) the normal vectors are not perpendicular to the corresponding surface anymore which distorts the lighting.

The trick of fixing this behavior is to use a different model matrix specifically tailored for normal vectors. This matrix is called the normal and uses a few linear algebraic operations to remove the effect of wrongly scaling normal vectors. 

The normal matrix is defined as the "transpose of the inverse of the upper-left 3x3 part of the model matrix".

Done worry if you don't understand this.

Most resources define the normal matrix as derived from the model-view matrix, but since we're working in world space, and not view space, we will derive it from the model matrix. 

In the vertex shader we can generate the normal matrix by using the inverse and transpose functions in the vertex shader that work on any matrix type. Note that we cast the matrix to a 3x3 matrix to ensure it loses its translation properties and that i can multiply with the vec3 normal vector. 

`Normal = mat3(transpose(inverse(model))) * aNormal;`

Inversing matrices is a costly operation for shaders, so whenever possible try to avoid doing inverse operations since they have to be done on each vertex of your scene. For learning purposes, this is fine, but for an efficient applications you'll likely want to calculate the normal matrix on the CPU and send it to the shaders via a uniform before drawing. 

In the diffuse lighting section the lighting was fine because we didn't do any scaling on the object, so there was not really a need to use a normal matrix and we could've just multiplied the normals with the model matrix. If you are doing a non-uniform scale however, it is essential that you multiply your normal vectors with the normal matrix. 

**Specular Lighting**

Similar to diffuse lighting, specular lighting is based on the light's direction vector and the object's normal vectors. However, it is also based on the player's view direction and how they are looking at the object. 

Specular lighting is based on the reflective properties of surfaces, the highlight is casted wherever we would see the light reflected on the surface. You can see it via this following diagram. 


![[Pasted image 20250513160403.png]]

We calculate a reflection vector by reflecting the light direction around the normal vector. Then we calculate the angular distance between this reflection vector and the view direction. The closer the angle between them, the greater the impact of the specular light. The resulting effect is that we see a bit of a highlight when we're looking at the light's direction reflected via the surface. 

The view vector is one extra variable we need for specular lighting which we can calculate using the viewer's world space position and the fragment's position. Then, we calculate the specular's intensity, multiply this with the light color and add this to the ambient and diffuse components. 

We chose to do the lighting calculations in world space, but most people tend to prefer doing lighting in view space. An advantage of view space is that the viewer's position is always at (0, 0, 0) so you already got the position of the viewer. However, I find calculating lighting in world space more intuitive for learning purposes. If you still want to calculate lighting in view space you want to transform all the relevant vectors with the view matrix as well (don't forget to change the normal matrix as well). 

To get the world space coordinates of the view we simply take the position vector of the camera object. So let's add another uniform to the fragment shader and pass the camera position to the shader. 

`uniform vec3 viewPos;'

In the C++ main file
`lightingShader.setVec3("viewPos", camera.Position);`

Now that we have all the required variables we can calculate the specular intensity. First we define a specular intensity value to give the specular highlight a medium-bright color so that it doesn't have too much impact:

`float specularStrength = 0.5;`

If we set this to 1.0f, it would be a really bright highlight. 

Next, we calculate the view direction vector and the corresponding reflect vector along the normal axis:

`vec3 viewDir = normalize(viewPos - FragPos);`
`vec3 reflectDir = reflect(-lightDir, norm);`

Note that we negate the lightDir vector. 

The reflect function expects the first vector point from the light source towards the fragment's position, but the lightDir is currently pointing the other way around: from the fragment towards the light source (this depends on how you calculated the light distance earlier). To make sure we get the correct reflect vector we reverse its direction by adding a negative to the lightDir vector first. The second argument expects a normal vector so we supply the normalized norm vector. 

Then what's left to do is to actually calculate calculate the specular component. This is accomplished with this formula.

`float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);`
`vec3 specular = specularStength * spec * lightColor;`

We first calculate the dot product between the view direction and the reflect direction (and make sure it's not negative using the max function) and then raise it to the power of 32. This 32 value is the shininess value of the highlight. The higher the shininess value of an object, the smaller and more condensed the highlight is. 

Here is a image of the different outcomes depending on how high you raise the power of to create the spec var. 

![[Pasted image 20250513165229.png]]

We don't want the specular component to be too distracting so we keep the exponent at 32. The only thing left to do is to add it to the ambient and diffuse components and then multiply that sum by the object color. Then use that result as the output color. 

Developers prior used to implement phong lighting in the vertex shader. The advantage of this is that it is a lot more efficient since there are generally a lot less vertices compared to fragments, so the expensive lighting calculations are done less frequently. However, the resulting color value in the vertex shader is the resulting lighting color of that vertex only and the color values of the surrounding fragments are then the result of interpolated lighting colors. The result was that the lighting was not very realistic unless large amounts of vertices where used.

When phong lighting is implemented in the vertex shader its called Gouraud shading. Note that due to the interpolation the lighting looks somewhat off. 

![[Pasted image 20250513170827.png]]

**Materials**

In the real world, each object has a different reaction to light. For example, steel objects are shinier than clay or wooden objects. Some objects reflect the light without much scattering resulting in smaller specular highlights and others scatter a lot giving the highlight a larger radius. If we want to simulate different types of objects in OpenGL we have to define material properties specific to each surface. 

In the previous chapter we defined an object and light color to define the visual output of an object, combined with an ambient and specular intensity component. When describing a surface we can define a material color for each of the 3 lighting components: ambient, diffuse, and specular. By specifying a color for each of the components we have fine-grained control over the color output of the surface. Now add a shininess component of those 3 colors and we have all the material properties we need. 

`#version 330 core`
`struct Material {`
	`vec3 ambient;'`
	`vec3 diffuse;`
	`vec3 specular;`
	`float shininess;`
`};`

`uniform Material material;`

In the fragment shader we create a struct to store the material properties of a surface. We can also store them as individual values, but storing them as a struct keeps it more organized. We first define the layout of the struct and then simply declare a uniform variable with the newly created struct as its type. 

How a struct works is now with that uniform variable, we can assign all of those properties within that struct to that single variable. For example, to assign it the shininess we would do material.shininess = 0.5. 

As you can see, we define a color vector for each of the Phong's lighting components. The ambient material vector defines what color the surface reflects under ambient lighting; this is usually the same as the surface's color. The diffuse material vector defines the color of the surface under diffuse lighting. The diffuse color is just like ambient lighting set to the desired surface's color. The specular material vector sets the color of the specular highlight on the surfaces or possibly even reflect a surface specific color. Lastly, the shininess impacts the scattering/radius of the specular highlight. 

With these 4 components that define an object's material we can simulate many real-world materials. A table as found at http://devernay.free.fr/cours/opengl/materials.html shows some general material properties/values to simulate real materials. 

![[Pasted image 20250514111114.png]]

As you can see, by correctly specifying the material properties of a surface it seems to change the perception we have of the object. The effects are clearly noticeable, but for more realistic results we'll need to replace the cube with something more complicated. 

Figuring out the right material settings for an object is a difficult feat that mostly requires experimentations and a lot of experience. It's not that uncommon to completely destroy the visual quality of an object by a misplaced material. 

Let's try implementing such a material system in the shaders.

**Setting Materials**

We created a uniform material struct in the fragment shader so next we want to change the lighting calculations to comply with the new material properties. Since all the material variables are stored in a struct we can access them from the material uniform. 

A uniform variable is a variable that can be accessed from any other file and is not confined to that individual file.  Which is why we can set our variables from the main C++ file using the shader header file. 

`void main()`
`{`
	`//ambient`
	`vec3 ambient = lightColor * material.ambient;`

	`//diffuse`
	`vec3 norm = normalize(Normal);`
	`vec3 lightDir = normalize(lightPos - FragPos);`
	`float diff = (max(dot(norm, lightDir), 0.0);`
	`vec3 diffuse = lightColor * (diff * material.diffuse);`

	`//specular`
	`vec3 viewDir = normalize(viewPos - FragPos);`
	`vec3 reflectDir = reflect(-lightDir, norm);`
	`float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);`

	`vec3 result = ambient + diffuse + specular;`
	`FragColor = vec4(result, 1.0);`
	
`}`

As you can see we now access all of the material's struct properties wherever we need them and this time calculate the resulting output color with the help of the material's colors. Each of the object's material attributes are multiplied with their respective lighting components. 

We can set the material of the object in the main C++ file by setting the appropriate uniforms using the shader header file. 

A struct in GLSL however is not special in any regard when setting uniforms; a struct only really acts as a namespace of uniform variables. If we want to fill the struct we will have to set the individual uniforms, but prefixed with the struct's name. 

`lightingShader.setVec3("material.ambient", 1.0f, 0.5f, 0.31f);`
`lightingShader.setVec3("matieral.diffuse", 1.0f, 0.5f, 0.31f);`
`lightingShader.setVec3("material.specular", 0.5f, 0.5f, 0.5f);`
`lightingShader.setFloat("matieral.shininess", 32.0f);`

We set the ambient and diffuse component to the color we'd like the object to have and set the specular component of the object to a medium-bright color;  we don't want the specular component to be strong. We also keep the shininess at 32. 

We can now easily influence the object's material from the application. Running the program gives you something like this.

![[Pasted image 20250514145759.png]]

It doesn't really look right though?

**Light Properties**

The object is way too bright. The reason for the object being too bright is that the ambient, diffuse, and specular colors are reflected with full force from any light source. Light sources also have different intensities for their for their ambient, diffuse, and specular components respectively. In the previous chapter we solved this by varying the ambient and specular intensities with a strength value. We want to do something similar, but this time by specifying intensity vectors for each of the light components. If we'd visualize lightColor as vec3(1.0) the code would look like this.

`vec3 ambient = vec3(1.0) * material.ambient;`
`vec3 diffuse = vec3(1.0) * (diff * material.diffuse);`
`vec3 specular = vec3(1.0) * (spec * material.specular);`

So each material property of the object is returned with full intensity for each of the light's components. These vec3(1.0) values can be influenced individually as well for each light source and this is usually what we want. Right now the ambient components shouldn't really have such a big impact on the final color so we can restrict the ambient color by setting the light's ambient intensity to a lower value:

`vec3 ambient = vec3(0.1) * material.ambient;`

We can influence the diffuse and specular intensity of the light source in the same way. This closely similar to what we did previously. You could say we already created some light properties to influence each lighting component individually. We'll want to create something to the material struct for the light properties.

`struct Light {`
	`vec3 position;`

	`vec3 ambient;`
	`vec3 diffuse;`
	`vec3 specular;`
`};`

`uniform Light light;`

A light source has a different intensity for its ambient, diffuse, and specular components. The ambient light is usually set to a low intensity because we don't want the ambient color to be too dominant. The diffuse component of a light source is usually set to the exact color we'd like a light to have; often a bright white color. The specular component is usually kept at vec3(1.0) shining at full intensity. Note that we also added the light's position vector to the struct. 

Just like with the material uniform we need to update the fragment shader.

`vec3 ambient = light.ambient * material.ambient;`
`vec3 diffuse = light.diffuse * (diff * material.diffuse);`
`vec3 specular = light.specular * (spec * material.specular);`

We then want to set the light intensities in the application:

`lightingShader.setVec3("light.ambient", 0.2f, 0.2f, 0.2f);`
`lightingShader.setVec3("light.diffuse", 0.5f, 0.5f, 0.5f);`
`lightingShader.setVec3("light.specular", 1.0f, 1.0f, 1.0f);`

Now that we modulated how the light influences the object's material we get a visual output that looks much like the output from the previous writing. This time however we got full control over the lighting and the material of the object;

![[Pasted image 20250514155809.png]]

As you can see, a different light color greatly influences the object's color output. Since the light color directly influences what colors the object can reflect, it has a significant impact on the visual output.

We can easily change the light's colors over time by changing the light's ambient and diffuse colors via sin and glfwGetTime.

`glm::vec3 lightColor;`
`lightColor.x = sin(glfwGetTime() * 2.0f);`
`lightColor.y = sin(glfwGetTime() * 0.7f);`
`lightColor.z = sin(glfwGetTime() * 1.3f);`

`glm::vec3 diffuseColor = lightColor * glm::vec3(0.5f);`
`glm::vec3 ambientColor = diffuseColor * glm::vec3(0.2f);`

`lightingShader.setVec3("light.ambient", ambientColor);`
`lightingShader.setVec3("light.diffuse", diffuseColor);`

The glfwGetTime() constantly changing the value in real time. The float you are multiplying by is the speed the color is changing. Lastly we use the sign to keep the values bouncing in-between -1 and 1.

Try and experiment with several lighting and material values and see how they affect the visual output.

**Lighting Maps**

Previously, we discussed the possibility of each object having a unique material of its own that reacts differently to light. This is great for giving each object a unique look in comparison to other objects, but still doesn't offer much flexibility on the visual output of an object. 

Objects in the real world do not consist of a single material, but of several materials. Like a car which has metal, plastic, glass, and many other materials that react differently. The car can also have diffuse and ambient colors that are not the same for the entire object; a car displays many different ambient/diffuse colors. All in all, such an object has many different material properties for each of its different parts.

So the material system previously discussed isn't sufficient for all but the simplest models so we need to extend the system by introducing diffuse and specular maps. These allow us to influence the diffuse (and indirectly the ambient component since they should be same anyways) and the specular component of an object with much more precision. 

**Diffuse Maps**

What we want is some way to set the diffuse colors of an object for each individual fragment. Some sort of system where we can retrieve a color value based on the fragment's position on the object?

This should probably all sound familiar and we've been using such a system for a while now. This sounds just like textures and it is basically just that: a texture. We're just using a different name for for the same underlying principle: using an image wrapped around an object that we can index for unique color values per fragment. In lit scenes this is usually called a diffuse map (this is generally how 3d artists call them before PBR) since a texture image represents all the object's diffuse colors. 

Using a diffuse map in shaders is exactly like using a texture. This time however, we store the texture as a sampler2d inside the Material struct. We replace the earlier defined vec3 diffuse color vector with a diffuse map,

Keep in mind that sampler2d is a so called opaque type which means we can't instantiate these types, but only define them as uniforms. If the struct would be instantiated other than as a uniform like as a function parameter/argument, GLSL would throw weird errors. The same applies to any struct holding such opaque types.

We also remove the ambient material color vector since the ambient color is equal to the diffuse color anyways now that we control ambient with the light. So there's no need to store it separately.

`struct Material {`
	`sampler2D diffuse;`
	`vec3 specular;`
	`float shininess;`
`};`
`...`
`in vec2 TexCoords;`

If you're a bit stubborn and still want to set the ambient colors to a different value (other than the diffuse value) you can keep the ambient vec3, but then the ambient colors would still remain the same for the entire object. To get different ambient values for each fragment you'd have to use another texture for ambient values alone. 

Note that we are going to need texture coordinates again in the fragment shader, so we declared an extra input variable. Then we simply sample from the texture to retrieve the fragment's diffuse color value. 

`vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));`

Also, don't forget to set the ambient material's color equal to the diffuse material's color as well. 

`vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));`

And that's all it takes to use a diffuse map. As you can see it is nothing new, but it does provide a dramatic increase in visual quality. To get it working we do need to update the vertex data with texture coordinates, transfer them as vertex attributes to the fragment shader, load the texture, and bind the texture to the appropriate texture unit. 

The vertex data now includes vertex positions, normal vectors, and texture coordinates for each of the cube's vertices. Let's update the vertex shader to accept texture coordinates as a vertex attribute and forward them to the fragment shader. 

`#version 330 core`
`layout (location = 0) in vec3 aPos;`
`layout (location = 1) in vec3 aNormal;`
`layout (location = 2) in vec2 aTexCoords;`
`...`
`out vec2 TexCoords;`

`void main()`
`{`
	`...`
	`TexCoords = aTexCoords;`
`}`

Be sure to update the vertex attribute pointers to both VAO's to match the new vertex data and load the container image as a texture. Before rendering the cube, we want to assign the right texture unit to the material.diffuse uniform sampler and bind the container texture to this unit.

`lightingShader.setInt("material.diffuse", 0);`
`...`
`glActivateTexture(GL_TEXTURE0);`
`glBindTexture(GL_TEXTURE_2D, diffuseMap);`

Now using a diffuse map we get an enormous boost in detail again and this time the container really starts to shine. Your object should look something like this. 

![[Pasted image 20250515111809.png]]

**Specular Maps**

You probably noticed that the specular highlight looks a bit odd since the object is a wooden container and wood doesn't have specular highlights like that. We can fix this by setting the specular material of the object to vec3(0.0) but that the steel borders of the container would stop showing specular highlights as well and steel should show specular highlights. We would like to control what parts of the object should show a specular highlights. We would like to control what parts of the object should show a specular highlight with varying intensity. This is a problem that sounds familiar.

We can also use a texture map just for specular highlights. This means we need to generate a black and white (or colors) texture that defines the specular intensities of each part of the object. Here is an example of a specular map:

![[Pasted image 20250515115142.png]]



The intensity of the specular highlight comes from the brightness of each pixel in the image. Each pixel of the specular map can be displayed as a color vector as a color vector where black represents the color vector vec3(0.0) and gray the color vector vec3(0.5) for example. In the fragment shader we then sample the corresponding color value and multiply this value with the light's specular intensity. The more white the pixel is, the higher the result of the multiplication and thus the brighter the specular component of an object becomes. 

Because the container mostly consists of wood, and wood as a material should have no specular highlights, the entire wooden section of the diffuse texture was converted to black: black sections do not have any specular highlight. The steel border of the container has varying specular intensities with the steel itself being relatively susceptible to specular highlights while cracks are not. 

Technically wood also has specular highlights although with a much lower shininess value (more light scattering)  and less impact, but for learning purposes we can just pretend wood doesn't have any reaction to specular light. 

Using tools like photoshop it is relatively easy to transform a diffuse texture to a specular image like the one above.

**Sampling Specular Maps**

A specular map is just like any other texture so the code is similar to the diffuse map code. Make sure to properly load the image and generate a texture object. Since we're using another texture sampler in the same fragment shader we have to use a different texture unit. for the specular map so let's bind it to the appropriate texture unit before rendering:

`lightingShader.setInt("material.specular", 1);`
`...`
`glActivateTexture(GL_TEXTURE1);`
`glBindTexture(GL_TEXTURE_2D, spcularMap);`

Then update the material properties of the fragment shader to accept a sampler2D as its specular component instead of vec3.

`struct Material {`
	`sampler2D diffuse;`
	`sampler2D specular;`
	`float shininess;`
`};`

And lastly we want to sample the specular map to retrieve the fragment's corresponding specular intensity.

`vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
`vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));`
`vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));`
`FragColor = vec4(ambient + diffuse + specular, 1.0);`

By using a specular map we can we can specify enormous detail what parts of an object have shiny properties and we can even control the corresponding intensity. Specular maps give us an added layer of control over lighting on top of the diffuse map. 

If you don't want to be too mainstream you could also use actual colors in the specular map to not only set the specular intensity of each fragment, but also the color of the specular highlight. Realistically however, the color of the specular highlight is mostly determined by the light source itself so it wouldn't generate realistic visuals. This is why the images are usually black and white we only care about the intensity.

![[Pasted image 20250515122056.png]]


**Light Casters**

All the lighting we've used so far came from a single source that is a single point in space. A light source that casts light upon objects is called a light caster. In this section, we'll discuss several different types of light casters. Learning to simulate different light sources is yet another tool in your toolbox to further enrich your environments. 

We'll first discuss a directional light, then a point light which is an extension of what we had prior, and lastly we'll discuss spotlights. Later on we will combine these light sources into one light scene. 

**Directional Light**

When a light source is far away the light rays coming from the light source are close to parallel to each other. It looks like all the light rays are coming from the same direction, regardless of where the object and/or viewer is. When a light source is modeled to be infinitely far away it is called a direction light since all its light rays have the same direction; it is independent of the location of the light source. 

A fine example of a direction light is the sun. The sun is not infinitely far away from us, but it is so far away that we can perceive it as being infinitely far away in lighting calculations. All the light rays from the sun are then modeled as parallel light rays as we can see in the following image. 


![[Pasted image 20250515130723.png]]

Because all the light rays are parallel it does not matter how each object relates to the light source's position since the light direction remains the same for each object in the scene. Because the light's direction vector stays the same, the lighting calculations will be similar for each object in the scene. 

We can model such a directional light by defining a light direction vector instead of a position vector. The shader calculations will be similar for each object in the scene.

We can model such a directional light by defining a light direction vector instead of a position vector. The shader calculations remain mostly the same except this time we directly use the light's direction instead of calculating the lightDir vector using the light's position vector.

`struct Light {`
	`// vec3 position; // no longer needed when using directional lights`
	`vec3 direction;`

	`vec3 ambient;`
	`vec3 diffuse;`
	`vec specular;`
	
`};`
`...`
`void main()`
`{`
	`vec3 lightDir = normalize(-light.direction);
	`...`
`}`

Note that we first negate the light.direction vector. The lighting calculations we used so far expect the light direction to be a direction from the fragment towards the light source. But people generally prefer to specify a direction light as a global direction pointing from the light source. Therefore, we need to negate have to negate global light direction vector to switch its direction; it's now a direction vector pointing towards the light source. Also, be sure to normalize the vector since it is unwise to assume that input vector to be a unit vector. 

The resulting lightDir vector is then used as before in diffuse and specular computations. 

To clearly demonstrate that a directional light has the same effect on multiple objects we revisit the container party scene from a previous OpenGL lesson. Which was 10 different cubes with textures on it spread around at different positions in the application. 

Here is a for loop for drawing those cubes:

`for(unsigned int = 0; i < 10; i++)`
`{`
	`glm::mat3 model = glm::mat4(1.0f);`
	`model = glm::translate(model, cubePositions[i])`
	`float angle = 20.0f * i`
	`model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));`
	`lightingShader.setMat4("model", model);`

	`glDrawArrays(GL_TRIANGLES, 0, 36);`
`}`

Also, don't forget to actually specify the direction of the light source (note that we define the direction as a direction from the light source;  you can quickly see the light's position is pointing downwards).

`lightingShader.setVec3("light.direction", -0.2f, -1.0f, -0.3f);`

We've been passing the light's position and direction vectors as vec3 for a while now, but some people tend to prefer to keep all the vectors defined as vec4. When defining position vectors as a vec4 it is important to set the w component to 1.0 so translation and projections are properly applied. However, when defining a direction vector as a vec4 we don't want translations to have an effect (since they just represent direction, nothing more) so then we define the w component to be 0.0.

Direction vectors can be represented as `vec4(-0.2f, -1.0f, -0.3f, 0.0f)`. This can also function as an easy check for light types: you could check if the w component is equal to 1.0 to see that we now have a light's position vector and if w is equal to 0.0 we have a light's direction vector; so adjust the calculations based on that:

`if(lightVector.w == 0.0) // note: be careful for floating point errors`
	`// do directional light calculations`
`else if(lightVector.w == 1.0)`
	`// do light calculations using the light's position`

If you'd now compile the application and move through the scene it looks like a sun-based light source casting light on all the objects. Can you see that the diffuse and specular components all react as if there was a light source somewhere in the sky?


![[Pasted image 20250515141355.png]]


**Point Lights**

Directional lights are great for global lights that illuminate the entire scene, but we usually also want several point lights scattered throughout the scene. A point light is a light source with a given position somewhere in a world that illuminates in all directions, where the light rays fade out over distance. Think of light bulbs and torches as light casters that act as a point light. 

![[Pasted image 20250515142026.png]]


We have worked with simplistic point light. We had a light source at a given position that scatters light in all directions from that given light position. However, the light source we defined simulated light rays that never fade out thus making it look like the light source is extremely strong. In most 3d applications we'd like to simulate a light source that only illuminates an area close to the light source and not the entire scene. 

If you'd add the 10 containers to the lighting scene, you'd notice that the container all the way in the back is lit with the same intensity as the container in the front of the light; there is no logic yet that diminishes the light over distance. We want the container in the back to only be slightly lit in comparison to the containers close to the light source.

**Attenuation**

To reduce the intensity of light cover over the distance a light ray travels is generally called attenuation. One way to reduce the light intensity over distance is to simply use a linear equation. Such an equation would linearly reduce the light intensity over the distance thus making sure that objects at a distance are less bright. However, such a linear function tends to look a bit fake. In the real world, lights are generally quite bright standing close by, but the brightness of a light source diminishes quickly at a distance; the remaining light intensity then slowly diminishes over distance. We are thus in need of a different equation for reducing the light's intensity. 

Luckily some smart people already figured this out for us. The following formula calculates an attenuation value based on a fragment's distance to the light source which we later multiply with the light's intensity vector. 

`Fatt = 1.0/Kc + Kl * d + Kq * d2`

Here d represents the distance from the fragment to the light source. Then to calculate the attenuation value we define 3 (configurable) terms: a constant term Kc, a linear term Kl, and a quadratic term Kq. 

The constant term is usually kept at 1.0 which is mainly there to make sure the denominator never gets smaller than 1 since it would otherwise boost the intensity with certain distances, which is not the effect we're looking for.

The linear term is multiplied with the distance value that reduces the intensity in a linear fashion. 

The quadratic term is multiplied with the quadrant of the distance and sets a quadratic decrease of intensity for the light source. The quadratic term will be less significant compared to the linear term when the distance is small, but gets much larger as the distance grows.

Due to the quadratic term the light will diminish mostly at a linear fashion until the distance becomes large enough for the quadratic term to surpass the linear term and then the light intensity will decrease a lot faster. The resulting effect is that the light is quite intense when at close range, but quickly loses its brightness over distance until it eventually loses its brightness at a more slower pace. The following graph shows the effect such an attenuation has over a distance of 100. 

![[Pasted image 20250515150632.png]]


You can see that the light has the highest intensity when the distance is small, but as soon as the distance grows its intensity is is significantly reduced and slowly reaches 0 intensity at around a distance of 100. This is exactly what we want. 

**Choosing The Right Values**

But at what values do we set those 3 terms? Setting the right values depend on many factors: the environment, the distance you want to cover, the type of light etc. In most cases, it simply is a question of experience and a moderate amount of tweaking. The following table shows some of the values these terms could take to simulate a realistic light source that covers a specific radius. The first column specifies the distance a light will cover with these given terms. These values are good starting points for most lights.

|Distance|Constant|Linear|Quadratic|
|---|---|---|---|
|`7`|`1.0`|`0.7`|`1.8`|
|`13`|`1.0`|`0.35`|`0.44`|
|`20`|`1.0`|`0.22`|`0.20`|
|`32`|`1.0`|`0.14`|`0.07`|
|`50`|`1.0`|`0.09`|`0.032`|
|`65`|`1.0`|`0.07`|`0.017`|
|`100`|`1.0`|`0.045`|`0.0075`|
|`160`|`1.0`|`0.027`|`0.0028`|
|`200`|`1.0`|`0.022`|`0.0019`|
|`325`|`1.0`|`0.014`|`0.0007`|
|`600`|`1.0`|`0.007`|`0.0002`|
|`3250`|`1.0`|`0.0014`|`0.000007`|

As you can see, the constant term Kc is kept at 1.0 in all cases. The linear term Kl is usually quite small to cover over larger distances and the quadratic term Kq is even smaller. Try to experiment a bit with these values to see their effect in your implementations. In our environment a distance of 32 to 100 is generally enough for most lights. 

**Implementing Attenuation**

To implement attenuation we'll be needing 3 extra values in the fragment shader: namely the constant, linear and quadratic terms of the equation. These are best stored in the Light struct we defined earlier. Note that we need to calculate lightDir again using position as this is a point light and not a directional light. 

`struct Light {`
	`vec3 position;`
	
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;

	float constant;
	float linear;
	float quadratic;


`};`



Then we set the terms in the main C++ file: we want the light to cover a distance of 50 so we'll use the appropriate constant, linear, and quadratic terms from the table:

`lightingShader.setFloat("light.constant", 1.0f);`
`lightingShader.setFloat("light.linear", 0.09f);`
`lightingShader.setFloat("light.quadratic", 0.032f);`

Implementing attenuation in the fragment shader is relatively straightforward: we simply calculate an attenuation value based on the equation and multiply this with the ambient, diffuse, and specular components. 

We do need the distance to the light source for the equation to work with though. Remember how we can calculate the length of a vector? We can retrieve the distance term by calculating the difference vector between the fragment and the light source and take that resulting vector's length. We can use GLSL's built-in **length** function for that purpose.

`float distance = length(light.position - FragPos);`
`float attenuation = 1.0 / (light.constant + light.linear * distance + light.quadratic * (distance * distance));`

Then we include this attenuation value in the lighting calculations by multiplying the attenuation value with the ambient, diffuse and specular colors. 

We could leave the ambient component alone so ambient lighting is no decreased over distance, but if we were to use more than 1 light source all the ambient components will start to stack up. In that case we want to attenuate ambient lighting as well. Simply play around with what's best for your environment. 

`ambient *= attenuation;`
`diffuse *= attenuation;`
`specular *= attenuation;`

If you run the application you'd get something like this. 

![[Pasted image 20250515155535.png]]

You can see that right now only the front containers are lit with the closest container being the brightest. The containers in the back are not lit at all since they're too far from the light source. 


**Spotlight**

The last type of light we're going to discuss is a spotlight. A spotlight is a light source that is located somewhere in the environment that, instead of shooting light rays in all directions, only shoots them in a specific direction. The result is that only the objects within a certain radius of the spotlight's direction are lit and everything else stays dark. A good example of a spotlight would be a street lamp or a flashlight. 

A spotlight in OpenGL is represented by a world-space position, a direction and a cutoff angle that specifies the radius of the spotlight. For each fragment we calculate if the fragment is between the spotlight's cutoff direction (in its cone) and if so, we lit the fragment accordingly. The following image gives you an idea of how a spotlight works.


![[Pasted image 20250515160359.png]]

LightDir (black): the vector pointing from the fragment to the light source.

SpotDir (red): the direction the spotlight is aiming at.

Phi ϕ (blue): the cutoff angle that specifies the spotlights radius. Everything outside this angle is not lit by the spotlight. 

Theta θ (green): the angle between the LightDir (black) vector and the SpotDir (red) vector. The theta θ (green) value should be smaller than the Phi Φ (blue) to be inside the spotlight. 

So what we basically need to do, is calculate the dot product (returns the cosine of the angle between between two vectors) between the LightDir vector and the SpotDir vector and compare this with the cutoff angle Phi (Blue). Now that you understand what a spotlight is all all about we're going to create one in the form of a flashlight. 

**Flashlight**

A flashlight is a spotlight located at the viewer's position and usually aimed straight ahead from the player's perspective. A flashlight is basically a normal spotlight, but with its position and direction continually updated based on the player's position and orientation. 

So, the values we're going to need for the fragment shader are the spotlight's position vector (to calculate the fragment-to-light's direction vector), the spotlight's direction vector and the cutoff angle. We can store these values in the Light struct:

`struct Light {`
	`vec3 position;`
	`vec3 direction;`
	`float cutOff;`
	`...`
`};`

Next we pass the appropriate values to the shader:

`lightingShader.setVec3("light.position", camera.Position);`
`lightingShader.setVec3("light.direction", camera.Front);`
`lightingShader.setFloat("light.cutOff", glm::cos(glm::radians(12.5f)));`

As you can see we're not setting an angle for the cutoff value but calculate value the cosine value based on an angle and pass the cosine result to the fragment shader. The reason for this is that in the fragment shader we're calculating the dot product between the LightDir and the SpotDir and the dot product returns a cosine value and not an angle; and we can't directly compare an angle with a cosine value. To get the angle in the shader we then have to calculate the inverse cosine of the dot product's result which is an expensive operation. 

Now what's left to do is calculate the theta (green) value and compare this with the cutoff value to determine if we're in or outside the spotlight:

`float theta = dot(lightDir, normalize(-light.direction));`

`if(theta > light.cutOff)`
`{`
	`// do lighting calculations`
`}`
`else // else, use ambient light so scene isn't completly dark outside` 
	`color = vec4(light.ambient * vec3(texture(material.diffuse, TexCoords)), 1.0);`

We first calculate the dot product between the lightDir vector and the negated direction vector (negated, because we want the vectors to point towards the light source, instead of from). Be sure to normalize all the relevant vectors. 

You may be wondering wy there is a > sign instead of a < sign in the if guard. Shouldn't theta be smaller than the light's cutoff value to be inside the spotlight? That is right, but don't forget angle values are represented as cosine values and an angle of 0 degrees is represented as the cosine value of 1.0 while an angle of 90 degrees is represented as the cosine value of 0.0 as you can see here. 

![[Pasted image 20250515164301.png]]


You can now see that the closer the cosine value is to 1.0 the smaller its angle. Now it makes sense why theta needs to be larger than the cutoff value. The cutoff value is currently set at the cosine of 12.5 which is equal to 0.976 so a cosine theta value between 0.976 and 1.0 would result in the fragment being lit as if inside the spotlight. 

Running the application results in a spotlight that only lights the fragments that are directly inside the cone of the spotlight.

![[Pasted image 20250515164757.png]]

We will now soften up the light to make it more realistic. 

**Smooth/Soft Edges**

To create the effect of a smoothly-edged spotlight we want to simulate a spotlight having an inner and an outer cone. We can set the inner cone as the cone defined previously. But we also want an outer cone that gradually dims the light from the inner to the edges of the outer cone. 

To create an outer cone we simply define another cosine value that represents the angle between the spotlight's direction vector, and the outer cone's vector (equal to its radius). Then, if a fragment is between the outer cone it should calculate an intensity value between 0.0 and 1.0. If the fragment is inside the inner cone its intensity is equal to 1.0 and if the fragment is outside the outer cone it will be 0.0.

We can calculate this with the following equation. 

I = θ−γ / ϵ

Here ϵ (epsilon) is the cosine difference between the phi (green) and the outer cone (γ) (ϵ = phi (green) - γ).

The resulting I value is then the intensity of the spotlight at the current fragment. 

It is a bit hard to visualize this formula actually works so let's try it out with a few sample values:

| θ       | θ in degrees | ϕ (inner cutoff) | ϕ in degrees | γ (outer cutoff) | γ in degrees | ϵ                         | I                               |
| ------- | ------------ | ---------------- | ------------ | ---------------- | ------------ | ------------------------- | ------------------------------- |
| `0.87`  | `30`         | `0.91`           | `25`         | `0.82`           | `35`         | `0.91 - 0.82 = 0.09`      | `0.87 - 0.82 / 0.09 = 0.56`     |
| `0.9`   | `26`         | `0.91`           | `25`         | `0.82`           | `35`         | `0.91 - 0.82 = 0.09`      | `0.9 - 0.82 / 0.09 = 0.89`      |
| `0.97`  | `14`         | `0.91`           | `25`         | `0.82`           | `35`         | `0.91 - 0.82 = 0.09`      | `0.97 - 0.82 / 0.09 = 1.67`     |
| `0.83`  | `34`         | `0.91`           | `25`         | `0.82`           | `35`         | `0.91 - 0.82 = 0.09`      | `0.83 - 0.82 / 0.09 = 0.11`     |
| `0.64`  | `50`         | `0.91`           | `25`         | `0.82`           | `35`         | `0.91 - 0.82 = 0.09`      | `0.64 - 0.82 / 0.09 = -2.0`     |
| `0.966` | `15`         | `0.9978`         | `12.5`       | `0.953`          | `17.5`       | `0.9978 - 0.953 = 0.0448` | `0.966 - 0.953 / 0.0448 = 0.29` |

As you can see we're basically interpolating between the outer cosine and the inner cosine based on the theta (blue) value. If you still don't really see what's going on, don't worry, you can simply take the formula for granted and return here when you're much older and wiser. 

We now have an intensity value that is neither negative when outside the spotlight, higher than 1.0 when inside the inner cone, and somewhere in between around the edges. If we properly clamp the values we don't need an if-else in the fragment shader anymore and we can simply multiply the light components with the calculated intensity value.

`float theta = dot(lightDir, normalize(-light.direction));`
`float epsilon = light.cutOff - light.outerCutOff;`
`float intensity = clamp((theta - light.outerCutOff) / epsilon, 0.0, 1.0);`
`...`
`// we'll leave ambient unaffected so we always have a little light`
`diffuse *= specular;`
`specular *= intensity;`
`...`

Note that we use the clamp function that clamps its first argument between the values 0.0 and 1.0. This makes sure the intensity values won't end up outside that range.

Make sure you add the outerCutOff value to the Light struct and set its uniform value in the application.

![[Pasted image 20250515170555.png]]

**Multiple Lights**

In this section we will talk about combining all lighting methods discussed previously into one fully lit scene. We are going to simulate a sun using a directional light source, 4 point lights around the scene as well as a flashlight. 

To use more than one light source in the scene we want to encapsulate the lighting calculations into GLSL functions. The reason for that is that the code quickly gets nasty when we do lighting computations with multiple light types, each requiring different computations. If we were to do all these calculations in the main function only, the code becomes difficult to understand. Functions in GLSL are just like C-functions. We have a function name, a return type and we need to declare a prototype at the top of the code file if the functions hasn't been declared yet before the main function. We'll create a different function for each light type: directional lights, point lights, and spot lights. 

When using multiple lights in a scene the approach is usually as follows: we have a single color vector that represents the fragment's output color. For each light, the light's contribution to the fragment is added to this output color vector. So each light in the scene will calculate its individual impact and contribute that to the final output color. 

`out vec4 FragColor;`

`void main()`
`{`
	`// define an output color value`
	`vec3 output = vec3(0.0);`
	`// add the directional light's cotribution to the output`
	`output += someFunctionToCalculateDirectionalLight();`
	`// do the same for all point lights`
	`for(int i = 0; i < nr_of_point_lights; i++)`
		`output += someFunctionToCallPointLight();`
	`// and add other lights as well (like spotlights)`
	`output += someFunctionToCalculateSpotLight();`

	`FragColor = vec4(output, 1.0);`

`}`

The actual code will likely differ per implementation, but the general structure remains the same. We define several functions that calculate the impact per light source and add its resulting color to an output color vector. If for example, two light sources are close to the fragment, their combined contribution would result in a more brightly lit fragment compared to the fragment being like being lit by a single light source. 

**Directional Light**

We want to define a function in the fragment shader that calculates the contribution a directional light has on the corresponding fragment: a function that takes a few parameters and returns the calculated direction lighting color. 

First we need to set the required variables that we minimally need for a directional light source. We can store those variables in a struct called DirLight and declare it as a uniform. The struct's variables should be familiar. 

`struct DirLight {`
	`vec3 direction;`

	`vec3 ambient;`
	`vec3 diffuse;`
	`vec3 specular;`
`};`
`uniform DirLight dirLight;`

We can then pass the dirLight uniform to a function with the following prototype.

`vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir);`

Just like C and C++, when we want to call a function (in this case the main function) the function should be defined somewhere before the caller's line number. In this case we'd prefer to define the functions below the main function so this requirement doesn't hold. Therefore we declare the function's prototypes somewhere above the main function, just like we would in C.

You can see that the function requires a DirLight struct and two other vectors required for its computation. If you successfully completed the previous chapter then the content of this function should come as no surprise:

`vec3 CalcDirLight(Dirlight light, vec3 normal, vec3 viewDir)`
`{`
	`vec3 lightDir = normalize(-light.direction);`
	`// diffuse shading`
	`float diff = max(dot(normal, lightDir), 0.0);`
	`// specular shading`
	`reflectDir = reflect(-lightDir)`
	`float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);`
	`// combine results`
	`vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));`
	`vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse));`
	`vec3 specular = light.specular * spec * vec3(texture(material.specular,                                                                    TexCoords));`
	`return (ambient + diffuse + specular);`
`}`

We basically copied the code from the previous chapter and used the vectors given as function arguments to calculate the direction light's contribution vector.  The resulting ambient, diffuse and specular contributions are then return as a single color vector. 

**Point Light**

Similar to directional lights we also want to define a function that calculates the contribution a point light has on the given fragment, including its attenuation. Just like directional lights we want to define a struct that specifies all the variables required for a point light. 

`struct PointLight {`
	`vec3 position;`

	`float constant;`
	`float linear;`
	`float quadratic;`

	`vec3 ambient;`
	`vec3 diffuse;`
	`vec3 specular;`
`};`
`#define NR_POINT_LIGHTS 4`
`uniform PointLight pointLights[NR_POINT_LIGHTS];`

As you can see we used a pre-processor directive in GLSL to define the number of point lights we want to have in our scene. We then use this NR_POINT_LIGHTS constant to create an array of PointLight structs. Arrays in GLSL are just like C arrays and can be created by the use of two square brackets. Right now we have 4 PointLight structs to fill with data. 

The prototype of the point light's function is as follows:

`vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir);`

The function takes all the data it needs as its arguments and returns a vec3 that represents the color contribution that this specific point light has on the fragment. 

`vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir)`
`{`
	`vec3 lightDir = normalize(light.position - fragPos);`
	`// diffuse shading`
	`float diff = max(dot(normal, lightDir), 0.0);`
	`// specular shading`
	`vec3 reflectDir = reflect(-lightDir, normal);`
	`float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);`
	`// attenuation`
	`float distance = length(light.position - fragPos);`
	`float attenuation = 1.0 / (light.constant + light.linear * distance + light.quadratic * (distance * distance));`

	`// combine results`

	`vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));`
	`vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse,                                                                         TexCoords));`
	`vec3 specular = light.specular * spec * vec3(texture(material.speccular,                                                                         TexCoords));`
	`ambient *= attenuation;
	`diffuse *= attenuation;`
	`specular *= attenuation;`

	`return (ambient + diffuse + specular);`
`}`

Abstracting this functionality away in a function like this has the advantage that we can easily calculate the lighting for multiple point lights without the need for duplicated code. In the main function we simply create a loop that iterates over the point light array that calls CalcPointLight for each point light. 

**Putting It All Together**

Now that we defined both a function for directional lights and a function for point lights, we can put it all together in the main function. 

`void main()`
`{`
	`// properties`
	`vec3 norm = normalize(Normal);`
	`vec3 viewDir = normalize(viewPos - FragPos);`

	`// phase 1: Directional Lighting`
	`vec3 result = CalcDirLight(dirLight, norm, viewDir);`
	`// phase 2: Point Lights`
	`for(int i = 0; i < NR_POINT_LIGHTS; i++)`
		`result += CalcPointLight(pointLights[i], norm, FragPos, viewDir);`
	`// phase 3: Spot Light`
	`//result += CalcSpotLight(spotLight, norm, FragPos, viewDir);`

	`FragColor = vec4(result, 1.0);`
`}`

Each light type adds its contribution to the resulting output color until all light sources are processed. The resulting color contains the color impact of all the light sources in the scene combined. We leave the CalcSpotLight function as an exercise. 

There are lot of duplicated calculations in this approach spread out over the light type functions (e.g. calculating the reflect vector, diffuse and specular terms, and sampling the material textures) so there's room for optimization here.

Setting uniforms for the directional light struct shouldn't be unfamiliar, but you may be wondering how to set the uniform values of the point lights since the point light uniform is actually an array of PointLight structs.

Setting the uniform values of an array of structs works like setting the uniforms of a single structs, although this time we also have to define the appropriate index when querying the uniform's location.

`lightingShader.setFloat("pointLights[0].constant", 1.0f);`

Here we index the first PointLight struct in the pointLights array and internally retrieve the location of its constant variable, which we set to 1.0.

Let's not forget that we also need to define a position vector of each of the 4 point lights so let's spread them up a bit around the scene. We'll define another glm::vec3 array that contains the pointlights' positions. 

`glm::vec3 pointLightPositions[] = {`
	`glm::vec3(0.7f, 0.2f, 2.0f),`
	`glm::vec3(2.3f, -3.3f, -4.0f),`
	`glm::vec3(-4.0f, 2.0f, -12.0f),`
	`glm::vec3(0.0f, 0.0f, -3.0f);`

`};`


Then we index the corresponding PointLight struct from the pointLights array and set its position attribute as one of the positions we just defined. Also be sure to now draw 4 light cubes instead of just 1. 

Simply create a different model matrix for each of the light objects just like we did with the containers. 


![[Pasted image 20250516164252.png]]


You can tweak the lighting values to get different environmental results. 


![[Pasted image 20250516164328.png]]


Also change the background with the clear color GLFW function to match the atmosphere. 

