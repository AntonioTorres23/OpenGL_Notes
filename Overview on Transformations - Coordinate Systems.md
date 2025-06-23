
This is just an overview of the LearnOpenGL topics regarding vectors, matrices, and how we apply them to graphics programming. 

**Vectors**

Vectors are simply directions and nothing more. A vector has a direction and a magnitude (AKA length). You can think of vectors like a point on a map. Vectors have can work with 2 to 4 dimensions. If a vector has 2 dimensions it represents a direction on an (X,Y) plane and if a vector has 3 dimensions (X,Y,Z), it can represent any direction in a 3D world. 

Below there are 3 vectors where each vector is in 2D (X,Y) as arrows on a graph. Because it is more intuitive to display vectors in 2D rather than 3D you can think of the vectors as 3D but with a z coordinate of 0. Since vector represent directions, the origin of the vector does not change its value. In the graph below we can see that the vectors v and w are equal even though their origin is different. 

This graph is kind of confusing since its describing the vectors within as translations in terms of how many it is moving over from its original dot point to move to a new position. Not in regards to its actual position on grid. 

![[Pasted image 20250623142902.png]]

When describing vectors people generally prefer to describe vectors as characters with a line over them. But since I have issues with that using a regular text editor like obsidian I am just going to describe them using a basic character like this for example: v = (X,Y,Z).  I know this all sounds kind of simple since we've been working ahead of this for a while now with using models and lighting and all that, however, just think of them as a `glm::vec3` variable in terms of actual application to something like a C++ file. So think of it as `glm::vec3 v (x,y,z);` in which (X,Y,Z) are float values.  



