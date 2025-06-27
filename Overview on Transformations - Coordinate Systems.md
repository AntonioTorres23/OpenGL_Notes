
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

A matrix is a rectangular array of numbers, symbols/or mathematical expressions. Each individual item in a matrix is called an **element** of the matrix. An example of a 2x3 matrix is shown below. 

$$  
\begin{bmatrix}  
1 & 2 & 3 \\  
4 & 5 & 6 \\  
\end{bmatrix}  
$$

Matrices are index by (i, j) where **i** is the **row** and **j** is the **column**, this is why the above matrix is called a 2x3 matrix (2 rows and 3 columns) (remember columns are vertical and columns are horizontal). These are also called the dimensions of the matrix. 

This is the opposite of what you're used to when indexing 2D graphs as (x, y). To retrieve the value 4 we would index it as (2,1) (second row, first column). 

Matrices are basically nothing more than that, just rectangular arrays of mathematical expressions. They do have a very nice set of mathematical properties and just like vectors we can define several operations on matrices, namely addition, subtraction, and multiplication. 

**Addition and Subtraction**

Matrix addition and subtraction between two matrices is done on a per-element basis. So the same general rules apply that we're familiar with for normal numbers, but done on the elements of both matrices with the same index (or position within the matrix). This does mean that addition and subtraction is only defined for matrices of the same dimensions. A 3x2 matrix and a 2x3 (or a 3x3 matrix and a 4x4 matrix). So you can only add and subtract from matrices that have the same numbers of rows and columns. 

For our examples of addition and subtraction lets use 2x2 matrices for our example. 

$$  
\begin{bmatrix}  
1 & 2 \\  
3 & 4 \\  
\end{bmatrix}  

+

\begin{bmatrix}  
5 & 6 \\  
7 & 8 \\  
\end{bmatrix}

=

\begin{bmatrix}  
1+5 & 2+6 \\  
3+7 & 4+8 \\  
\end{bmatrix}  

=
\begin{bmatrix}  
6 & 8 \\  
10 & 12 \\  
\end{bmatrix}

$$

The same rules apply for matrix subtraction.

$$
\begin{bmatrix}  
4 & 2 \\  
1 & 6 \\  
\end{bmatrix}

-

\begin{bmatrix}  
2 & 4 \\  
0 & 1 \\  
\end{bmatrix}

=

\begin{bmatrix}  
4-2 & 2-4 \\  
1-0 & 6-1 \\  
\end{bmatrix}

=
\begin{bmatrix}  
2 & -2 \\  
1 & 5 \\  
\end{bmatrix}
$$


**Matrix-scalar Products**

A matrix-scalar product multiples each element of the matrix by a scalar. The following example illustrates the multiplication. 

$$

2
⋅
\begin{bmatrix}  
1 & 2 \\  
3 & 4 \\  
\end{bmatrix}
=
\begin{bmatrix}  
1⋅2 & 2⋅2 \\  
3⋅2 & 4⋅2 \\  
\end{bmatrix}

=

\begin{bmatrix}
2 & 4 \\
6 & 8 \\
\end{bmatrix}
$$


Now it also makes sense as to why those single numbers are called scalars. A scalar basically scales all the elements of the matrix by its value. In the previous example, all elements were scaled by an integer of 2. 

**Matrix-matrix Multiplication**

Multiplying matrices is not necessarily complex, but rather difficult to get comfortable with. Matrix multiplication basically means to follow a set of pre-defined rules when multiplying. There are a few restrictions though.

1. You can only multiply two matrices if the number of columns on the left-hand side matrix is equal to the number of rows on the right-hand side matrix. 
2. Matrix multiplication is not **commutative** which means A ⋅ B ≠ B ⋅ A. 

We will show the matrix multiplication of 2 2x2 matrices. 
$$
\begin{bmatrix}  
1 & 2 \\  
3 & 4 \\  


\end{bmatrix}

⋅

\begin{bmatrix}  
5 & 6 \\  
7 & 8 \\  
\end{bmatrix}

=

\begin{bmatrix}  
1 ⋅ 5 + 2 ⋅ 7 & 1 ⋅ 6 + 2 ⋅ 8 \\  
3 ⋅ 5 + 4 ⋅ 7 & 3 ⋅ 6 + 4 ⋅ 8 \\  
\end{bmatrix}

=

\begin{bmatrix}
19 & 22 \\
43 & 50 \\
\end{bmatrix}
$$

So here's what I think is going on personally in my own thoughts. On the left hand side you take the row and multiply it with the same element position. So far example the 1st element of 1. You multiply that with the right hand matrix's 1st position element which is 5. Then we add this with another product. This is the 2nd element within the row on the left-hand which is 2 and the 3rd element in the first column which is 7. So that creates 1 ⋅ 5 + 2 ⋅ 7 = 19. This concept applies to the other elements within the matrices. There is a combination of multiplication and then adding those products together. This image with color highlighting helps with showing what is going on. 

![[Pasted image 20250624155946.png]]

See how our 1st row on the left-hand matrix matches with the 1st column on the right-hand matrix. As well as the 2nd row of the left-hand matrix matches with the 2nd column right-hand matrix. 

So position (1,1) of the left-hand matrix you would multiply by the position (1,1) of the right-hand matrix. But we also add the product of the element within the position of (1,2) on the left-hand matrix and the element within the position (2,1) on the right-hand matrix. This gives us our new value for the position (1,1) when we do matrix multiplication for the left hand matrix and right-hand matrix. Remember the element's position goes by row and then column where row is **i** and column is **j** (i , j). 

We go by first row and first column where we go by first row on the left-hand side matrix and first column on the right-hand side matrix. 

After your first row is done, you use the first row still and then use the second column. So you will take the product of the element in position (1,1) within the left-hand matrix and the element in position (1,2) in the right-hand matrix. Then you add it with the product of the element in position (1,2) in the left-hand matrix and element in the position (2,2) in the right-hand matrix. This creates the new value for the element within position (1,2). 

Now we move onto the 2nd row on the left-hand matrix but the 1st column on the right-hand matrix. So element in position (2,1) on the left-hand matrix will be multiplied with element in position (1,1) in the right-hand matrix. This will be added with the product of the element in the position of (2,2) on the left-hand matrix and the element in the position of (2,1) on the right-hand matrix. This gives us our new value for the element within position (2,1). 

Lastly we are still using the second row on the left-hand matrix but now we are using the 2nd column within our right-hand matrix. So we take the product of the element in position (2,1) within the left-hand matrix and the element in position (1,2) in the right-hand matrix. Then we add this by the product of the element in position (2,2) in the left-hand matrix and the element in position (2,2) within the right-hand matrix. This gives us our final value for the position (2,2) in our new matrix. 

Just remember left-hand matrix uses rows and right-hand matrix uses columns. Rows go left to right and columns go up to down. As well as remember when you are calculating the new matrix element value within a matrix position like (1,1) for example. You need to take the sum of two products. The first product being the row and the second one being the column. It makes more sense when you look at it visually so don't forget to look at that color-coded example above. 

However I overall think its best to calculate in the order from (1,1), (1,2), (2,1), (2,2) to make it less confusing overall. At least for me a lot of the times I was doing it from up and down rather than left to right which made me more confused. 

See how we need to have two matrices that have the same amount of rows and columns. 

Here is a more complex matrix multiplication with 2 3x3 matrices. 

![[Pasted image 20250625100410.png]]


As you can see, matrix multiplication is often confusing and is why its often done by computer. 

**Matrix-Vector Multiplication**

Up until now we've used vectors quite often. We use them to represent positions, colors, and even texture coordinates. Let's move a bit further down the rabbit hole and tell you that a vector is basically a Nx1 matrix where **N** is the vector's number of components (also known as an **N-dimensional** vector). If you think about it, it makes a lot of sense. Vectors are just like matrices an array of numbers, but with only 1 column. So, how does this new piece of information help us? Well, if we have a `MxN` matrix we can multiply this matrix with our `Nx1` vector, since the columns of the matrix are equal to the number of rows of the vector, thus matrix multiplication is defined. 

But why do we care if we can multiply matrices with a vector? Well, it just so happens that there are lots of interesting 2D/3D transformations we can place inside a matrix, and multiplying that matrix with a vector then transforms that vector. In case you're confused, let's start with a few examples and you'll soon see what we mean. 

**Identity Matrix**

In OpenGL we usually work with 4x4 transformation matrices for several reasons and one of them is that most vectors are of size 4. The most simple transformation matrix we can think of is the **identity matrix**. The identity matrix is an `NxN` matrix with only 0s except on its diagonal. As you'll see, this transformation matrix leaves a vector completely unharmed. 

My theory on this is that each row represents the 4 points on a vector that being row 1 is x, row 2 is y, row 3 is z, and row 4 is w or your transformation component which is often normalized to the number of 1. It's basically a hidden point on every vector that processes things like scaling or translations.

As well as it goes left to right like what we see within your average vector format so row one position (1,1) is x so all other elements within this row are now 0 since we can't have two or more x values within one vector. This repeats on each row reflecting the position of a traditional vector like instead of position (1,1) for x. You do position (2,2) for y, then position (3,3) for z, lastly position (4,4) for w. Like a traditional 4 value vector, (x , y, z, w) you go left to right with each column reflecting the vector value position. 

Here is an image example:

![[Pasted image 20250625134232.png]]

The vector is completely untouched. This becomes obvious from the rules of multiplication: the first result element is each individual element of the first row of the matrix multiplied with each element of the vector. Since each row of the elements are 0 except for the first one, we get: 
$$
(1) ⋅ 1 + (0) ⋅ 2 + (0) ⋅ 3 + (0) ⋅ 4 = 1
$$
and the same applies for the other 3 elements of the 4 value vector. 

You may be wondering what the use is of a transformation matrix that does not transform? The identity matrix is usually a starting point for generating other transformation matrices and if we dig even deeper into linear algebra, a very useful matrix for proving theorems and solving linear equations. 

**Scaling**

When we're scaling a vector we are increasing the length of the arrow by the amount we'd like to scale, keeping its direction the same. Since we're working in either 2 or 3 dimensions we can define scaling by a vector of 2 or 3 scaling variables, each scaling one axis (x, y, or z). 

Let's try scaling the vector v = (3,2). We will scale the vector along the x-axis by 0.5, thus making it twice as narrow; we'll scale the vector by 2 along the y-axis, making it twice as high. Let's see what it looks like if we scale the vector by (0.5, 2) as vector s. 

![[Pasted image 20250625165013.png]]

Keep in mind that OpenGL usually operates in 3D space so far this 2D case we could set the z-axis scale to 1, leaving it unaffected. The scaling operation we just preformed is a **non-uniform scale**, because the scaling factor is not the same for each axis. Meaning that you have different scaling values on all the (x, y, z, w) positions. If the scalar would be equal on all axes it would be called a **uniform scale**. Meaning you would gave all the same scaling value applied to the (x, y, z, w) vector positions. 

Let's start building a transformation matrix that does the scaling for us. We saw from the identity matrix that each of the diagonal element were multiplied with the corresponding vector element. What if we were to change the 1s in the identity matrix to 3s? In that case, we would be multiplying each of the vector elements by a value of 3 and thus preforming a uniform scale. If we represent the scaling variables as (S1, S2, S3) we can define a scaling matrix on any vector (x, y, z) as

![[Pasted image 20250625165950.png]]

Note that we keep the 4th scaling value 1. The w component is used for other things such as keeping the magnitude or length of a vector as one. 

**Translation**

**Translation** is the process of adding another vector on top of the original vector to return a new vector with a different position, thus, moving the vector based on a translation vector. We've already discussed vector addition so this shouldn't be new. 

Just like the scaling matrix there are several locations on a 4x4 matrix that we can use to perform certain operations and for translation those are the top-3 values of the 4th column. If we represent the translation vector as (T1, T2, T3) we can define the translation matrix by.

$$
\begin{bmatrix}
1 & 0 & 0 & Tx \\
0 & 1 & 0 & Ty \\
0 & 0 & 1 & Tz \\ 
0 & 0 & 0 & 1 \\
\end{bmatrix}

⋅

\begin{pmatrix}
x\\
y\\
z\\
1\\ 
\end{pmatrix}

=

\begin{pmatrix}
x + Tx \\
y + Ty \\
z + Tz \\
1
\end{pmatrix}
$$

Also use this color coded diagram for reference:

![[Pasted image 20250626095811.png]]

This works because all of the translation values are multiplied by the vector's w column and added to the vector's original values (remember the matrix-multiplication rules). This wouldn't have been possible with a 3x3 matrix. 

I think in regards to the matrix multiplication think of the identity matrix. In which we multiply each row going from 1st element within the matrix and the 1st element within the vector but instead the vector is in a column. So for the x coordinate within the translation it would be: 

$$
(1) ⋅ x + (0) ⋅ y + (0) ⋅ z + (Tx) ⋅ 1 = x + 0 + 0 + Tx \\
$$
As you can see, 0 times whatever the value of y or z is just 0 so in theory to prevent further confusion you can just leave them out. Which is why the equations above just have `x + Tx` or `y + Ty`, or `z + Tz`

So here is what all those equations would look like for each row of the matrix multiplied with the columns of the vector for the equations/diagrams above.

Translated X Value: 
$$
(1) ⋅ x + (0) ⋅ y + (0) ⋅ z + (Tx) ⋅ 1 = x + 0 + 0 + Tx \\
$$

Translated Y Value:
$$
(0) ⋅ x + (1) ⋅ y + (0) ⋅ z + (Ty) ⋅ 1 = y + 0 + 0 + Ty \\
$$

Translated Z Value:
$$
(0) ⋅ x + (0) ⋅ y + (1) ⋅ z + (Tz) ⋅ 1 = 0 + 0 + z + Tz \\
$$
Translated W Value (Doesn't Change):
$$
(0) ⋅ x + (0) ⋅ y + (0) ⋅ z + (1) ⋅ 1 = 0 + 0 + 0 + 1 \\
$$



I knew my hypothesis was right for the sky box when we get the 3x3 matrix of the camera view and put that into a 4x4 because we are eliminating the existing 4th column values with something like 0 which prevents the skybox from moving with the camera view so it always looks far away.

**Homogenous Coordinates**

The w component is also known as a homogenous coordinate. To get the 3D vector from a homogenous vector we divide the x, y, and z coordinate by its w coordinate. We usually do not notice this since the w component is 1.0 most of the time. Using homogenous coordinates has several advantages. It allows us to do matrix translations on 3D vectors (without the w component we can't translate vectors). 

Also whenever the homogenous coordinate is equal to 0, the vector is specifically known as a direction vector since a vector with a w coordinate of 0 cannot be translated. 

With a translation matrix we can move objects in any of the 3 axis directions (x, y, z), making it a very useful transformation matrix for our transformation toolkit. 

For additional practice lets do this with some actual values, lets say we have a vector with the coordinates `(1,2,3)` and we want to move the x coordinate by 2, the y coordinate by 1, and the z coordinate by 3. 

Well we would simply add those values as our top 3 4th column elements within the matrix and do some matrix multiplication. Here is the equation down below. 


$$
\begin{bmatrix}
1 & 0 & 0 & 2 \\
0 & 1 & 0 & 1 \\
0 & 0 & 1 & 3 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}

\cdot

\begin{pmatrix}
1 \\
2 \\
3 \\
1
\end{pmatrix}

= 
\begin{bmatrix}
1 \cdot 1 + 0 \cdot 2 + 0 \cdot 3 + 2 \cdot 1\\
0 \cdot 1 + 1 \cdot 2 + 0 \cdot 3 + 1 \cdot 1 \\
0 \cdot 1 + 0 \cdot 2 + 1 \cdot 3 + 3 \cdot 1 \\
0 \cdot 1 + 0 \cdot 2 + 0 \cdot 3 + 1 \cdot 1 \\
\end{bmatrix}
=
\begin{bmatrix}
1 + 0 + 0 + 2 \\
0 + 2 + 0 + 1 \\ 
0 + 0 + 3 + 3 \\
0 + 0 + 0 + 1 \\
\end{bmatrix}

= 
\begin{pmatrix}
3 \\
3 \\ 
6 \\ 
1
\end{pmatrix}
= 

(3,3,6,1)

$$

So those translations would lead to the position being moved to the vector : (3,3,6) with one being your w coordinate that never changes. 

This is just to get a more in depth understanding of what is going on under the hood of general graphics programming and how these concepts are eventually applied to OpenGL. 

**Rotation**

The last few transformations were relatively easy to understand and visualize in 2D or 3D space, but rotations are a bit more difficult. 

First let's define what a rotation of a vector actually is. A rotation in 2D or 3D is represented with an angle. An angle could be in degrees or radians where a whole circle has 360 degrees or 2 $\pi$ radians. I prefer explaining rotations using degrees as we're generally more accustomed to them. 

Most rotation functions require an angle in radians, but luckily degrees are easily converted into radians. 

angle in degrees = angle in radians * (180 / $\pi$)

angle in radians = angle in degrees * ($\pi$ / 180)

Where $\pi$ equals (rounded) 3.14159265359.

Rotating half a circle rotates us 360/2 = 180 degrees and rotating 1/5th to the right means we rotate 360/5 = 72 degrees to the right. This is demonstrated for a basic 2D vector where v is rotated 72 degrees to the right, or clockwise from k.

![[Pasted image 20250626131737.png]]

Rotations in 3D are specified with an angle and a rotation axis. The angle specified will rotate the object along the rotation axis given. Try to visualize this by spinning your head a certain degrees continually looking down a single rotation axis. When rotating 2D vectors in a 3D world for example. We set the rotation axis to the z axis. 

Using trigonometry it is possible to transform vectors to newly rotated vectors given an angle. This is usually done via a smart combination of the sine and cosine functions (commonly abbreviated to sin and cos). 

A rotation matrix is defined for each unit in 3D space where the angle is represented as the theta symbol $\theta$. 

Rotation around the X-axis:

![[Pasted image 20250626133247.png]]

Rotation around the Y-axis:

![[Pasted image 20250626133520.png]]

Rotation around the Z-axis:

![[Pasted image 20250626133723.png]]

Using the rotation matrices we can transform our position vectors around on the three unit axes. To rotate around an arbitrary 3D axis we combine all 3 of them by rotating around the X-axis, then the Y-axis, and then the Z-axis for example. However, this quickly introduces a problem called **Gimbel Lock**. We won't discuss details but a better solution is to rotate around an arbitrary unit axis e.g. (0.662, 0.2, 0.722) (note that this is a unit vector) right away instead of combining the rotation matrices. Such a verbose matrix exists and is given below with (Rx, Ry, and Rz) as the arbitrary rotation axis.

![[Pasted image 20250626134935.png]]

Keep in mind that even this matrix does not completely prevent gimbal lock although it gets a lot harder. To truly prevent gimbal locks we have to represent rotations using quaternions, that are not only safer, but also more computationally friendly. 

**Combining Matrices**

The true power from using matrices for transformations is that we can combine multiple transformations in a single matrix thanks to matrix-matrix multiplication. Let's see if we can generate a transformation matrix that combines several transformations. Say we have a vector (x, y , z) and we want to scale it by 2 and then translate it by (1, 2, 3). We need a translation and a scaling matrix for our required steps. The resulting transformation matrix would then look like. 

![[Pasted image 20250626135604.png]]


Note that we first do a translation and then a scale transformation when multiplying matrices. Matrix multiplication is not commutative, which means their order is important. When multiplying matrices, the right-most matrix is first multiplied with the vector so you should read the multiplications from right to left. It is advised to first do scaling operations, then rotations and lastly translation when combining matrices otherwise they may (negatively) affect each other. For example, if you would first do a translation and then scale, the translation vector would also scale. 

Running the final transformation matrix on our vector results in the following vector.

![[Pasted image 20250626141745.png]]

Great, the vector is first scaled by two and then translated by (1,2,3). 

**In Practice**

Now that we've explained all the theory behind transformations, its time to see how we can actually use this to our advantage. OpenGL does not have any form of matrix or vector knowledge built in, so we have to define our own mathematics classes and functions. In this book we'd rather abstract from all these tiny mathematical details and simply use a pre-made mathematics libraries.

Luckily, there is an easy-to-use and tailored-for-OpenGL mathematics library called GLM.

**GLM**

GLM stands for OpenGL Mathematics and is a header-only library, which means that we only have to include the proper header files and we're done; no linking and compiling necessary. GLM can be downloaded from their [website](https://glm.g-truc.net/0.9.8/index.html).

Copy the root directory of the header files into your includes folder and let's get rolling. 

Most of GLM's functionality that we need can be found in 3 header files that we'll include as follows:

```
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
```

Let's see if we can put our transformation knowledge to good use by translating a vector of (1, 0, 0) by (1, 1, 0) (note that we define it as a `glm::vec4` with its homogeneous coordinate set to 1.0)

```
glm::vec4 vec(1.0f, 0.0f, 0.0f, 1.0f);
glm::mat4 trans = glm::mat4(1.0f);
trans = glm::translate(trans, glm::vec3(1.0f, 1.0f, 1.0f));
vec = trans * vec;
std::cout << vec.x << vec.y << vec.z << std::endl;
```

We first define a 4 value vector called `vec` using `GLM`'s built-in vector class. Next we define a 4x4 matrix using `GLM`'s `mat4` class and explicitly initialize it to the identity matrix by initializing the matrix's diagonals (the 1st element position (1,1), 2nd element position (2,2), 3rd element position (3,3), and 4th position element (4,4)) to 1.0; if we do not initialize it to the identity matrix by the matrix would be a null matrix (all elements 0) and all subsequent matrix operations would end up as a null matrix as well.  

The next step is to create a transformation matrix passing our identity matrix to the `glm::translate` function, together with a translation vector (the given matrix is then multiplied with a translation matrix and the resulting matrix is returned). 

Then we multiply our vector by the transformation matrix and output the result. If we still remember how matrix translation works then the resulting vector should be ((1) + 1, (0) + 1, 0 + 0) which is (2, 1, 0). This snippet of code outputs 210 so the translation did its job. 

Let's do something more interesting and scale and rotate an object. 

```

glm::mat4 trans = glm::mat4(1.0f);
trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
trans = glm::scale(trans, glm::vec3(0.5, 0.5, 0.5));
```

First we scale the container by 0.5 on each axis and then rotate the container 90 degrees around the Z-axis. GLM expects its angles in radians so we convert the degrees to radians using the function `glm::radians`. Note that the textured rectangle is on the XY plane so we want to rotate around the Z-axis. Keep in mind that the axis that we rotate around should be a unit vector, soo be sure to normalize the vector first if you're not rotating around the X, Y,  or Z axis. Because we pass the matrix to each of GLM's functions, GLM automatically multiples the matrices together, resulting in a transformation matrix that combines all the transformations. 

The big question is: how do we get the transformation matrix to the shaders? We shortly mentioned before that GLSL also has a `mat4` type. So we'll adapt the vertex shader to accept a `mat4` uniform variable and multiply the position vector by the matrix uniform.

```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

uniform mat4 transform;

void main()
{
	gl_Position = transform * vec4(aPos, 1.0f); // This is where we multiply the                                                      transformation matrix by the                                                       actual position vertices that                                                      are where the object is.
	TexCoord = vec2(aTexCord.x, aTexCoord.y);
}
```


GLSL also has `mat2` (2x2 Matrices) and `mat3` (3x3 Matrices) types that allow for swizzling-like operations just like vectors. All the aforementioned math operations (like scalar-matrix multiplication, matrix-vector multiplication, and matrix-matrix multiplication) are allowed on matrix types. Wherever special matrix operations are used we'll explain what's happening. 


We added the uniform and multiplied the position vector with the transformation matrix before passing it to `gl_Position`. Our container should now be twice as small and rotated 90 degrees (tilted on its left). We still need to pass the transformation matrix to the shader though. 

```
unsigned int transformLoc = glGetUniformLocation(ourShader.ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```

We first query the location of the uniform variable and then send the matrix data to the shaders using `glUniform` with `Matrix4fv` as its postfix. The first argument should be familiar which is the uniforms location. The second argument tells OpenGL how many matrices we'd like to send which is 1. The third argument asks us if we want to transpose our matrix, that is to swap the columns and rows. OpenGL developers often use an internal matrix layout called column-major ordering which is the default matrix layout in GLM  so there is no need to transpose matrices; we can keep it at `GL_FALSE`. The last parameter is the actual matrix data, but GLM stores their matrices' data in a way that doesn't always match OpenGL's expectations so first convert the data with GLM's built in function `value_ptr`. 

We created a transformation matrix, declared a uniform in the vertex shader and sent the matrix to the shaders where we transform our vertex coordinates. The result should look something like this.

![[Pasted image 20250626170918.png]]


Perfect! Our container is indeed tilted to the left and twice as small so the transformation was successful. Let's get a little more funky and see if we can rotate the container over time, and for fun we'll also reposition the object at the bottom-right side of the window. To rotate the container overtime we have to update transformation matrix in the render loop because it needs to update each frame. We use GLFW's time function to get an angle over time.

```
glm::mat4 trans = glm::mat4(1.0f);
trans = glm::translate(trans, glm::vec3(0.5f, 0.5f, 0.0f));
trans = glm::rotate(trans, (float)glfwGetTime(), glm::vec3(0.0f, 0.0f, 1.0f));
```

Keep in mind that in the previous case we could declare the transformation matrix anywhere, but now we have to create it every iteration to continually update the rotation. This means we have to re-create the transformation matrix in each iteration of the render loop. Usually when rendering scenes we have several transformation matrices that are re-created with new values each frame. 

Here we first rotate the container around the origin (0, 0, 0) and once it's rotated, we translate its rotated version to the bottom-right corner of the screen. Remember that the actual transformation order should read in reverse: even though in code we first translate and then later rotate, the actual transformations first apply a rotation and then 