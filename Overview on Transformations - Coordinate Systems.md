
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

So `-v = -(1,1,2)`

**Addition and Subtraction**

Addition of two vectors is defined as a component-wise addition, that is each component of one vector is added to the same component of the other vector. 

So basically if you have 2 vectors `v = (x,y,z)` and `x = (x, y, z)`. You just do `(x+x), (y+y), (z+z)` and the sum of those points are your new vector.  

Same applies to subtraction but instead of adding you just subtract. 

Addition of 2 Vectors: 

![[Pasted image 20250623155717.png]]

Subtraction of 2 Vectors:

Disregard this graph its weird and will just confuse you more. 


![[Pasted image 20250623160021.png]]

**Length**

To retrieve the length/magnitude of a vector we use the Pythagoras theorem. A vector forms a triangle when you visualize its individual x and y component as two sides of a triangle. 

![[Pasted image 20250623160550.png]]\

So the length of the two sides (x and y) are known and we want to know the length of the titled side v, we can calculate it using the Pythagoras Theorem.

||v|| = $\sqrt{x^2 + y^2}$

In this case the length of vector (4,2) equals.

||v|| = $\sqrt{4^2 + 2^2}$ =  $\sqrt{20}$ = 4.47

Which means the length of vector v is equal to 4.47.

There is another type of vector called a unit vector. A unit vector has one extra property and that is that its length is exactly 1. We can calculate a unit vector n from any vector by dividing each of the vectors components by its length. 

n = v/||v|| 

We call this normalizing a vector. Unit vectors are displayed with a little roof over their head and are generally easier to work with, especially when we only care about directions (the direction does not change if we change a vector's length). 

**Vector-vector Multiplication**

Multiplying two vectors is a bit of a weird case. Normal multiplication isn't really defined on vectors since it has no visual meaning. but we have two specific cases that we could choose from when multiplying: one is the dot product denoted as v ⋅ k and the other is the cross product denoted as v x k. 

**Dot Product**

The dot product of two vectors is equal to the scalar product of their lengths times the cosine of the angle between them. 


v ⋅k =||v||⋅||k||⋅cosθ

Where the angle between them is represented as a theta (θ). Why is this interesting? Well, imagine if v and k are unit vectors then their length would be equal to 1. This would effectively reduce the formula to.

v ⋅ k= 1 ⋅ 1⋅ cosθ = cosθ

Now the dot product only defines the angle between both vectors. You may remember that the cosine or cos function becomes 0 when the angle is 90 degrees or 1 when the angle is 0. This allows us to easily test if the two vectors are orthogonal or parallel to each other using the dot product (orthogonal means the vectors are at a right-angle to each other). 

So how do we calculate a dot product? The dot product is a component-wise multiplication where we add the results together. It looks like this with two unit vectors.

`(x=0.6, y=-0.8, z=0) ⋅ (x=0, y = 1. z=0) = (0.6 * 0) + (-0.8 * 1), (0 * 0) = -0.8`

To calculate the degree between these unit vectors we use the inverse cosine function cos−1 and this results in 143.1 degrees. We now effectively calculated the angle between two vectors. The dot product proves very useful for lighting calculations as we learned in one the notes here. 

**Cross Product**

The cross product is only defined in 3D space and takes two non-parallel vectors as input and produces a third vector that is orthogonal (vectors are at a right-angle to each other) to both input vectors. If both the input vectors are orthogonal to each other as well, a cross product would result in 3 orthogonal vectors. The following picture shows what this looks like in 3D space.

![[Pasted image 20250623165806.png]]

Unlike the other operations the cross product isn't really intuitive without delving into linear algebra so it's best to just memorize the formula and you'll be fine. Below you'll see the cross product between two orthogonal vectors A and B. 

![[Pasted image 20250623170034.png]]

So I always think of it as you first start with the diagonal product on the left hand vector first, then for the next product that you subtract with the first you flip it. So with Ay and Bz the product you would subtract from would be Az * By. You just flip the axis. 

**Matrices**

A matrix is a rectangular array of numbers, symbols/or mathematical expressions. Each individual item in a matrix is called an element of the matrix. An example of a 2x3 matrix is shown below. 

$$  
\begin{bmatrix}  
1 & 2 & 3 \\  
4 & 5 & 6 \\  
\end{bmatrix}  
$$

Matrices are index by (i, j) where i is the row and j is the column, this is why the above matrix is called a 2x3 matrix (3 columns and 2 rows) (remember columns are vertical and columns are horizontal). 