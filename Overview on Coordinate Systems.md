

We learned how we can use matrices to our advantage by transforming all vertices with transformation matrices. OpenGL expects all vertices, that we want to become visible, to be in normalized device coordinates after each vertex shader run. That is, the x, y, and z coordinates of each vertex should be between the values of -1.0 and 1.0; coordinates outside this range will not be visible. What we usually do, is specify the coordinates in a range (or space) we determine ourselves and in the vertex shader transform these coordinates to be **normalized device coordinates (NDC)**. These NDC are then given to the rasterizer to transform them to 2D coordinates/pixels on your screen. 

Transforming coordinates to NDC is usually accomplished in a step-by-step fashion where we transform an object's vertices to several coordinate systems before finally transforming them to NDC. The advantage of transforming them to several immediate coordinate systems is that some operations/calculations are easier in certain coordinate systems will soon become apparent. There are a total of 5 different coordinate systems that are of importance to us:

- Local Space (or Object Space)
- World Space
- View Space (or Eye Space)
- Clip Space
- Screen Space

Those are all a different state at which our vertices will be transformed in before in before finally ending up as fragments. 

You're probably quite confused by now by what a space or coordinate system actually is so we'll explain them in a more high-level fashion by showing the total picture and what each specific space represents. 

**The Global Picture**

To transform the coordinates from one space to the next coordinate space we'll use several transformation matrices of which the most important are the **model**, **view**, and **projection** matrix (see why learning this is important). Our vertex coordinates first start in **local space** as **local coordinates** and are then further processed to **world coordinates**, **view coordinates**, **clip coordinates**, and eventually end up as **screen coordinates**. The following image displays the process and shows what each transformation does.


![[Pasted image 20250627104040.png]]


1. Local coordinates are the coordinates of your object relative to its local origin; they're the coordinates your object begins in. 
2. The next step is to transform the local coordinates to world-space coordinates which are coordinates in respect of a larger world. These coordinates are relative to some global origin of the world, together with many other objects also placed relative to this word's origin. 
3. Next we transform the world coordinates to view-space coordinates in such a way that each coordinate is as seen from the camera or from the camera or viewer's point of view. 
4. After the coordinates are in view space we we want to project them to clip coordinates. Clip coordinates are processed to the -1.0 and 1.0 range and determine which vertices will end up on the screen. Projection to clip-space coordinates can add perspective if using perspective projection. 
5. And lastly we transform the clip coordinates in a process we call **viewport transform** that transforms the coordinates from -1.0 and 1.0 to the coordinate range defined by `glViewport`. The resulting coordinates are then sent to the rasterizer to turn them into fragments. 

You probably got a slight idea what each individual space is used for. The reason we're transforming our vertices into all these different spaces is that some operations make more sense or are easier to use in certain coordinate systems. For example, when modifying your object it makes most sense to do this in local space, while calculating certain operations on the object with respect to the position of other objects makes most sense in world coordinates and so on. If we want, we could define one transformation matrix that goes from local space to clip space in one go, but that leaves us with less flexibility. 

We'll discuss each coordinate system in more detail below.

**Local Space**

Local space is the coordinate space that is local to your object i.e. where your object begins in. Imagine that you've created your cube in a modeling software package (like Blender). The origin of your cube is probably at (0, 0, 0) even though your cube may end up at a different location in your final application. Probably all the models you have created all have (0, 0, 0) as their initial position. All the vertices of your model are therefore in local space: they are all local to your object.

The vertices of the container we've been using were specified as coordinates between -0.5 and 0.5 with 0.0 as its origin. These are local coordinates. 

**World Space**

If we would import all our objects directly in the application they would probably all be somewhere positioned inside each other at the worlds origin (0, 0, 0) which is not what we want. We want to define a position for each object to position them inside a larger world. The coordinates in world space are exactly what they sound like: the coordinates of your object are transformed from local to world space; this is accomplished with the **model** matrix. 

The **model matrix** is a transformation matrix that translates, scales and/or rotates your object to place it in the world at a location/orientation they belong to. Think of it as transforming a house by scaling it down (it was a bit too large in local space), translating it to a suburbia town and rotating it a bit to the left on the y-axis so it fits neatly with the neighboring houses. You could think of the matrix in the previous chapter to position the container all over the scene as a sort of model matrix as well; we transformed the local coordinates of the container to some different place on the scene/world.

So model matrix takes all your raw vertices of your object and transforms it so that you can do translations, scaling, or rotations in world space. 

**View Space**

The view space is what people usually refer to as the **camera** of OpenGL (it is sometimes also known as **camera space** or **eye space**). The view space is the result of transforming your world-space coordinates to coordinates that are in front of the user's view. The view space is thus the space seen from the camera's point of view. This is usually accomplished with a combination of translations and rotations to translate/rotate the scene so that certain items are transformed to the front of the camera. These combined transformations are generally stored inside a **view matrix** that transforms world coordinates to view space. If you remember this is how we create our first person camera within our application. 

**Clip Space**

At the end of each vertex shader run, OpenGL expects the coordinates to be within a specific range and any coordinate that falls outside this ranged is clipped. Coordinates that are clipped and discarded, so the remaining coordinates will end up as fragments visible on your screen. This is also where **clip space** gets its name from. 

Because specifying all the visible coordinates to be within -1.0 and 1.0 isn't really intuitive, we specify our own coordinate set to work in and convert those back to NDC as OpenGL expects them.

To transform vertex coordinates from view to clip space we define a so called **projection matrix** that specifies a range of coordinates e.g. -1000 and 1000 in each dimension. The projection matrix then converts coordinates within this specified range to normalized device coordinates (-1.0, 1.0) (not directly, a step called Perspective Division sits in between). All coordinates outside this range will not be mapped between -1.0 and 1.0 and therefore will be clipped. With this range we specified in the projection matrix, a coordinate of (1250, 500, 750) would not be visible, since the x coordinate is out of range and thus gets converted to a coordinate higher than 1.0 in NDC and is therefore clipped. 

Note that if only a part of primitive e.g. a triangle is outside the clipping volume OpenGL will reconstruct the triangle as one or more triangles to fit inside the clipping range. 

This "viewing box" a projection matrix creates is called a **frustum** and each coordinate that ends up inside this frustum will end up on the user's screen. The total process to convert coordinates within a specified range to NDC easily mapped to 2D view space coordinates is called **projection** since the projection matrix projects 3D coordinates to the easy to map to 2D normalized device coordinates. 

Once all the vertices are transformed to clip space a final operation called **perspective division** is preformed where we divide the x, y, and z components of the position vector by the vector's homogeneous w component. Remember 