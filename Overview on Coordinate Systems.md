

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

You probably got a slight idea what each individual space is used for. The reason we're transforming our vertices into all these different spaces is that some operations make more sense or are easier to use 