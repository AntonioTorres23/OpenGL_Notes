
This is just an overview of the LearnOpenGL topics regarding vectors, matrices, and how we apply them to graphics programming. 

**Vectors**

Vectors are simply directions and nothing more. A vector has a direction and a magnitude (AKA length). You can think of vectors like a point on a map. Vectors have can work with 2 to 4 dimensions. If a vector has 2 dimensions it represents a direction on an (X,Y) plane and if a vector has 3 dimensions (X,Y,Z), it can represent any direction in a 3D world. 

Below there are 3 vectors where each vector is in 2D (X,Y) as arrows on a graph. Because it is more intuitive to display vectors in 2D rather than 3D you can think of the vectors as 3D but with a z coordinate of 0. Since vector represent directions, the origin of the vector does not change its value. In the graph below we can see that the vectors v and w are equal even though their origin is different. 

This graph is kind of confusing since its describing the vectors within as translations in terms of how many it is moving over from its original dot point to move to a new position. Not in regards to its actual position on the grid.  So since v's original position on the grid is (0,0) and the vector says (3,2), the point is moved 3 times on the x axis and 2 times on the y axis. So the new point from that original (0,0) point is (3,2). Now look at the w vector, it starts at (1,3). It uses the same translation as the v vector where it goes to the right 3 times on the x axis and up 2 times on the y axis. So this moves the w vector to (4,5). Lastly, the n vector is originally at vector (6,3) and moves negatively to the left 3 on the x vector 3 times bringing it to the new position vector (3,3). 

Pretty easy just think back to those basic geometry classes and how a point can be placed on a graph. 

![[Pasted image 20250623142902.png]]

When describing vectors people generally prefer to describe vectors as characters with a line over them. But since I have issues with that using a regular text editor like obsidian I am just going to describe them using a basic character like this for example: v = (X,Y,Z).  I know this all sounds kind of simple since we've been working ahead of this for a while now with using models and lighting and all that, however, just think of them as a `glm::vec3` variable in terms of actual application to something like a C++ file. So think of it as `glm::vec3 v (x,y,z);` in which (X,Y,Z) are float values.  

**Scalar Vector Operations**

A scalar is a single digit. When adding/subtracting/multiplying or dividing a vector with a scalar, we simply add/subtract/multiply or divide each element of the vector by the scalar. 

So vector `v = (1,2,3)`, and we are going to add it by value x. So we are going to apply that to all 3 coordinates on a 3D vector (X,Y,Z). `new_v = (1+x,2+x,3+x)`. So you are basically taking whatever value is x and applying it to each point on the vector with whatever arithmetic your are doing. 

**Vector Negation**

Negating a vector results in a vector in the reversed direction. 

