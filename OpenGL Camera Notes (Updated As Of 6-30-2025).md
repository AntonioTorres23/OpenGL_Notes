
Camera works in the sense that transforms all the vertex coordinates from the camera's perspective as the origin of the scene: the view matrix transforms all the world coordinates into view coordinates that are relative to the camera's position and direction. 

To provide further information we have previously worked with matrices and learned that the view matrix is what provides us with a way to view objects in a 3D space. Remember that the view matrix is different from the projection matrix and the projection matrix deals with clip space rather than only the camera and its position. 

The view matrix transfers the world coordinates into a format that the user can view within your graphics application.  

When we're talking about camera/view space we're talking about all the vertex coordinates as seen from the camera's perspective as the origin of the scene: the view matrix transforms all the world coordinates into view coordinates that are relative to the camera's position and direction. To define a camera we need its position in world space, the direction it's looking at, a vector pointing to the right and a vector pointing upwards from the camera. A careful reader may notice we're actually going to create a coordinate system with 3 perpendicular unit axes with the camera's position as its origin. 

So its like we're creating our own x, y, and z axes just for our camera I believe. 

![[Pasted image 20250701142315.png]]
**1. Camera Position**

First we define a initial camera position by providing a vector in the world space. This is where the camera is at within the scene. Think of it as where your player or user would be if they where walking around. 

`glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f); // This is the inital position of the camera in the world space. `

Don't forget that the positive z-axis is going through your screen towards you so if we want to move the camera backwards, we move along the positive z axis.

**2. Camera Direction**

**CAUTION**: I AM GOING TO SPLIT THIS SECTION UP INTO SOME PARTS BECAUSE THERE IS A PORTION OF THIS REGARDING A STATIC CAMERA IN WHICH IT'S NOT CONTROLLED BY THE USER AND ONE SECTION THAT INVOVLES IMPLEMENTING A "FLY STYLE" CAMERA. 

**NO USER CONTROL CAMERA DIRECTION**

The next vector required is the camera's direction e.g. at what direction it is pointing at. For now we let the camera point to the origin of our scene: (0, 0, 0). Remember that if we subtract two vectors from each other we get a vector that's the difference of these two vectors? Subtracting the camera position from the scene's origin vector thus results in the direction vector we want. For the view matrix's coordinate system we want it's z-axis to be positive because by convention (OpenGL) the camera points towards the negative z-axis we want to negate the direction vector. If we switch the subtraction order around we now get a vector point towards the camera's positive z-axis:

```
glm::vec3 carmeraTarget = glm::vec3(0.0f, 0.0f, 0.0f); // where we want our cam to                                                           point at
glm::vec3 carmeraDirection = glm::normalize(cameraPos - cameraTarget); // your                                                                               cam's
                                                                          z-axis
```

Keep note that the z axis is pointing toward the camera. If we have `cameraTarget - cameraPos` so (0,0,0) - (0,0,3), it would result in the direction vector (0,0,-3) which would be facing negatively on the z-axis. So by swapping that order we get `cameraPos - cameraTarget` or, (0,0,3) - (0,0,0) which results in the direction vector (0,0,3) which is facing positively on the z-axis. 

*THE NAME DIRECTION VECTOR IS NOT THE BEST CHOSEN NAME, SINCE IT IS ACTUALLY POINTING IN THE REVERSE OF WHAT IT IS TARGETING. SO ITS REALLY POINTING FROM THE CAMERA TARGET FROM THE CAMERA POSITION*


**3. Right Axis**

**NO USER CONTROL RIGHT AXIS**

The next vector that we need is a right vector that represents the positive x-axis of the camera space. To get the right vector we use a little trick by first specifying an up vector that points upwards (in world space). Then we do a cross product on the up vector and the direction from step 2. Since the result of a cross product is a vector perpendicular to both vectors, we will get a vector that point's in the positive x-axis direction (if we switch the order of the cross product we would get a vector that points in the negative x-axis direction).
```
glm::vec3 up = glm::vec3 (0.0f,1.0f,0.0f); // vec3 that points up in world space
                                              no realtion to camera
glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection)); // x-axis                                                                             of                                                                                 camera
```

**4. Up Axis**

**NO USER CONTROL UP AXIS**

Now that we have both the x-axis vector (right-axis) of the camera, and the z-axis of the camera (`cameraDirection` `vec3`). Retrieving the vector that points to the camera's y-axis is relatively easy: we take the cross product of the right and direction vector.

```
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight); // cam's up (y-axis)
```

With the help of the cross-product and a few tricks we were able to create all the vectors that form the view/camera space. For the more mathematically inclined readers, the process in known as the [Gram-Schmidt](http://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process) process in linear algebra. Using these camera vectors we now create a `LookAt` matrix that proves very useful for creating a camera. 

**Look At**

A great thing about matrices is that if you define a coordinate space using 3 perpendicular (or non-linear) axes you can create a matrix with those 3 axes plus a translation vector and you can transform any vector to that coordinate space by multiplying it with this matrix. This is exactly what the `LookAt` matrix does and now that we have 3 perpendicular axes and a position vector to define the camera space we can create our own `LookAt` matrix.

![[Pasted image 20250702110003.png]]

Where R is the right vector (cam's positive x-axis), U is the up vector (cam's positive y-axis), D is the Direction vector (cam's positive z-axis), and P is the camera's position vector. Note that rotation (left matrix) and translation (right matrix) parts are inverted (transposed and negated respectively) since we want to rotate and translate the world in the opposite direction of where we want the camera to move. Using this `LookAt` matrix as our view matrix effectively transforms all world coordinates to the view space we just defined. The `LookAt` matrix then does exactly what it says: it creates a view matrix that looks at a given target. 

Luckily for us, GLM already does all this work for us. We only have to specify a camera position, a target position and a vector that represents the up vector in world space (the up vector we used for calculating the right vector). GLM then creates the `LookAt` matrix that we can use as our view matrix.

```
glm::mat4 view;

view = glm::lookAt(glm::vec3(0.0f, 0.0f, 0.3f), // camera position vec3
				   glm::vec3(0.0f, 0.0f, 0.0f), // camera target vec3
				   glm::vec3(0.0f, 1.0f, 0.0f)); // vec3 pointing up in world space
```

The `glm::LookAt` function requires a position, target, and up vector respectively.

Before delivering into user input, let's get a little funky first by rotating the camera around our scene. We keep the target of the scene at (0, 0, 0). We use a little bit of trigonometry to create an x and z coordinate each frame that represents a point on a circle and we'll use these for our camera position. By re-calculating the x and y coordinate over time we're traversing all the points in a circle and thus the camera rotates around the scene. We enlarge this circle by a pre-defined radius and create a new view matrix each frame using GLFW's `glfwGetTime` function. 

```
const float radius = 10.0f;
float camX = sin(glfwGetTime()) * radius;
float camZ = cos(glfwGetTime()) * radius;
glm::mat4 view;
view = glm::lookAt(glm::vec3(camX, 0.0, camZ), // camera position vec3 
	   glm::vec3(0.0, 0.0, 0.0),               // camera target vec3                      glm::vec3(0.0, 1.0, 0.0));              // vec3 pointing up in world space
```

Your code when ran with this as your view matrix should be spinning around the scene in circles over time.

![[Pasted image 20250703125456.png]]


**Walk Around**

Swinging the camera around a scene is fun, but it's more fun to do all the movement ourselves. First we need to set up a camera system, so it is useful to define some camera variables at the top of our program. 

```
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f); // cameras world position
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f); // camera view direction
glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f); // vec3 pointing up in world                                                          space
```

The `LookAt` function now becomes:

`view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);`

The view matrix variable holds the look at function which takes 3 parameters, the position of the camera, the target at which the camera wants to point at. As well as a vector that to the positive up axis (y-axis) in world space.

Here is the definition that are for the `lookAt` function parameters that LearnOpenGL provides. 

1st position: Specifies the position of the camera in word coordinates.

2nd target: Specifies the target direction/position the camera should look at. 

3rd up: specifies a vector pointing in the positive y direction in world space used to create the right vector. This is usually set at (0.0, 1.0, 0.0).

This is all preformed within the camera header file. 

First we set the camera position to the previously defined `cameraPos`. The direction (target/view we want the camera to face) is the current position of the camera + the direction vector (`cameraFront` which is where our camera is facing) we just defined. This ensures that however we move, the camera keeps looking at the target direction. Let's play a bit with these variables by updating the `cameraPos` vector when we press some keys.

We already defined a `processInput` function to manage GLFW's keyboard input so let's add a few extra key ccommands.

```
void processInput(GLFWwindow *window) // takes in GLFW windows pointer variable
{
	...
	const float cameraSpeed = 0.05f // adjust accordingly
	if (glfwGetKey(window, GLFW_KEY_W) == GFLW_PRESS)
		cameraPos += cameraSpeed * cameraFront;
	if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
		cameraPos -= cameraSpeed * cameraFront;
	if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
		cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) *                   cameraSpeed;
	if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
		cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * 
		cameraSpeed;

}
```

Whenever we press one of the WASD keys, the camera's position is updated accordingly, If we want to move forwards or backwards we add or subtract the direction from the position vector scaled by some speed value. If we want to move sideways, we do a cross product to create a right vector and we move along the right vector accordingly. This creates the familiar strafe effect when using the camera.

So I think of it like this, you have your `cameraPos` vector and whenever in the while loop a w key is pressed that is going forward so it will take the scalar of `cameraSpeed` (in this case 0.05) and  multiply it by the `cameraFront` vector (the global vector being (0, 0, -1)). This gives us the vector (0,0,-0.05). Then we add this with the camera's current position vector, (which starts at (0, 0, 3)), so the equation would be (0, 0, 3) + (0, 0, -0.05) = (0, 0, 2.95). Remember that since the positive z-axis is pointing towards the camera, the smaller the z coordinate for the camera's position gets, the more we move up in world space. 

This also applies when pressing the s key but in reverse, we do our multiplication with the `cameraSpeed` scalar (0.05) and `cameraFront` (0, 0, -1) to get the vector (0, 0, -0.05). However, we now subtract our new vector with the `cameraPos` vector which starts at a default of (0, 0, 3). So if our camera was at the default vector and we wanted to do subtraction, the equation would be (0, 0, 3) - (0, 0, -0.5) = (0, 0, 3.05). Remember that when we subtract a negative value with a negative value we get a positive value. Also note that since the z-axis points towards the camera, the camera position is actually moving backwards in world space. 

The d key is a bit different, when it is pressed we do a cross product of `cameraFront` (0, 0, -1) which is where our camera is pointing at and a vector called `cameraUp` (0, 1, 0) that is pointing towards the positive y-axis in world space. Upon doing this cross product we create the positive x-axis (also known as right axis) of the camera which in this case is (1, 0, 0). Also note that we normalize this cross product to make sure that the `cameraFront` vector doesn't affect the speed of the camera based on the orientation of the camera (I know that's pretty confusing). Then we multiply this by our `cameraSpeed` scalar (0.05) which gives us the vector (0.05, 0, 0). Then we add this to the global `cameraPos` variable which by default has a value of (0, 0, 3). The equation for this would be (0, 0, 3) + (0.05, 0, 0) = (0.05, 0, 3). This moves the camera position to right on the positive x-axis within world space. 

The same thing applies to the a key is similar but instead of adding to `cameraPos` we will be subtracting. we do a cross product of our `cameraFront` (0, 0, -1) and  `cameraUp` vectors (0,1,0) which generate the camera positive x (also known as right) axis (1, 0, 0). We then multiply this by our `cameraSpeed` scalar (0.05) which gives us the vector (0.05, 0, 0). Finally, we subtract this vector by the `cameraPos` vector which by default is set at (0,0,3). This gives us the result: (0,0,3) - (0.05, 0, 0) = (-0.05, 0, 3). This moves the camera position to the left on the x-axis within world space. 

Remember that the every time we do this the vector `cameraPos` updates and stores these values. This is why I specify that the default value for it is (0, 0, 3). Depending on how many times you do this or how far back, forth, left, or right you go, those values will always be different. Think of the position (0, 0, 3) as a default space that the camera will spawn in at when the app is first ran.  


When the camera object is created, it has predefined values that are used for the lookat function. This is stored within a function called GetViewMatrix within the camera header file. 

Within the main file, this is stored within the view variable and called via the camera object that was created earlier in the function. 

When the camera object is created with the header file, the predefined vector variables get data entered into them. 

**Movement Speed**

Currently we used a constant value for movement speed when walking around. In theory this seems fine, but in practice people's machines have different processing powers and the result of that is that some people are able to render much more frames than others each second. Whenever a user renders more frames than another user he also calls `processInput` more often. The result is that some people move really fast and some really slow depending on their setup. When shipping your application you want to make sure it runs the same on all kinds of hardware. 

Graphics applications and games usually keep track of a `deltatime` variable that stores the time it took to render the last frame. We then multiply all velocities with this `deltaTime` value. The result is that when we have a large `deltaTime` in a frame, meaning that the last frame took longer than average, the velocity for that frame will also be a bit higher to balance it out. When using this approach it does not matter if you have a very fast or slow PC, the velocity of the camera will be balanced out accordingly so each user will have the same experience. 

To calculate the `deltaTime` value we keep track of 2 global variables and within each frame we then calculate the new `deltaTime` value for later use.

```
float deltaTime = 0.0f; // Time between current frame and last frame float 
float lastFrame = 0.0f; // Time of last frame

float currentFrame = glfwGetTime(); 
deltaTime = currentFrame - lastFrame; 
lastFrame = currentFrame;

```

Now that we have a `deltaTime` we can take it into account when calculating the velocities.

```
  
void processInput(GLFWwindow *window) 
{ 
	float cameraSpeed = 2.5f * deltaTime; 
	[...] 
}
```

Since we're using `deltaTime` the camera will now move at a constant speed of 2.5 units per second. 

Together with the previous section we should now have a much smoother and more consistent camera system for moving around the scene. 

If you are stuck check the [source code](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/7.2.camera_keyboard_dt/camera_keyboard_dt.cpp) from the LearnOpenGL website. 

**Look Around**

Only using the keyboard keys to move around isn't that interesting. Especially since we can't turn around making the movement rather restricted. That's where the mouse comes in.

To look around the scene we have to change the `cameraFront` (where the camera is pointing at) vector based on the input of the mouse. However, changing the direction vector based on mouse rotations is a little complicated and requires some trigonometry.

**Euler Angles**

Euler angles are 3 values that can represent any rotation in 3D, defined by Leonhard Euler somewhere in the 1700s. There are 3 Euler angles: **pitch**, **yaw**, and **roll**. The following image gives them more meaning. 

![[Pasted image 20250707113930.png]]

The **pitch** is the angle that depicts how much we're looking up or down as seen in the first image. The second image shows the **yaw** value which represents the magnitude we're looking to the left or to the right. The **roll** represents how much we roll as mostly used in space-flight cameras. Each of the Euler angles are represented by a single value with the combination of all 3 of them we can calculate any rotation vector in 3D.

**Pitch** is how up or down we are looking at an object in 3D and is controlled by the y coordinate. 

**Yaw** is how much we are looking left or right at an object in 3D and it is determined by the x coordinate. 

So using trigonometry, you can use the cos and sin to get the position of x and y. 


Let's start with a bit of a refresher and check the general right triangle case (with one side at a 90 degree angle). 



 ![[Pasted image 20250304162505.png]]
`

If we define the hypotenuse to be a length of 1, we know from trigonometry (`soh`, `cah` `toa`) that the adjacent side's length is $\cos x/h = cos = x/1 = \cos x$ and that the opposing side's length is $\sin y/h = \sin = y/1 = \sin y$. This gives us general formulas for retrieving the length in both the x and y on right triangles, depending on the given angle. Let's use this to calculate components of the direction vector (front vector I think).

Let's imagine this same triangle, but now looking at it from a top perspective with the adjacent and opposite sides being parallel to the scene's x and z axis (as if looking down from the y-axis).

![[Pasted image 20250304162642.png]]


If we visualize the yaw angle to be the counter-clockwise angle starting from the x side we can see that the length of the x side relates to $\cos(yaw)$. And similarly how the length of the z side relates $\sin(yaw)$.  
 
If we take this knowledge and a given yaw value we can use it to create a camera direction vector.

```
glm::vec3 direction;
direction.x = cos(glm::radians(yaw)); // Note that we convert the angle to radians                                                                            first
direction.z = sin(glm::radians(yaw));
```

This solves how we can get a 3D direction vector from a yaw value, but pitch needs to be included as well. Let's now look at the y axis as if we're sitting on the `x/z` plane.  

This is accomplished similarly to Yaw where x and z are meeting with y to get an angle. 

![[Pasted image 20250305111404.png]]

Similarly, from this triangle we can see that the direction's y component equals $\sin(pitch)$ so let's fill that in.

`direction.y = sin(glm::radians(pitch));`

However, from the pitch triangle we can also see the `xz` sides are influenced by $\cos(pitch)$ so we need to make sure this is also part of the direction vector. With this included we get the final direction vector as translated from yaw and pitch Euler angles.

```
direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch)); 
direction.y = sin(glm::radians(pitch)); 
direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
```

So think of this like this, we have 3 axes we work on right x, y, and z. We are creating a triangle within where x and z are orthogonal as well as when x/z are orthogonal to y. We first apply our sin and cos to the yaw because we want to do our left and right translations first. However, since this is a 3D space, x and z are also influenced by the pitch due to them being orthogonal to y. So we also need to apply their cos and sin that relates to pitch to their respective coordinates. Since y only reflects how up and down (pitch) we are in a 3D space we only need to get the sin of pitch. 

Also remember that yaw and pitch are represented as theta ($\theta$) of the right triangles that are created when the axes are orthogonal to each other. They are represented with degrees. So think of it as we are getting where the coordinate of our camera direction/target (x, y, or z) via the yaw and pitch angles. Which is why for x and z we need to multiply the $\cos/sin(yaw)$  with the $\cos(pitch)$ because those two also are apart of the right triangle that is made with the y-axis.

![[Pasted image 20250707144828.png]]

This gives us a formula to convert yaw and pitch values to a 3-dimensional directional vector that we can use for looking around. 

We've set up the scene world so everything's positioned in the direction of the negative z-axis. However, if we look at the x and z triangle we see that the $\theta$ of 0 results in the camera's direction vector to point towards the positive x-axis. To make sure the camera points towards the negative z-axis by default we can give the yaw a default value of a 90 degree **clockwise** rotation. Positive degrees rotate **counter-clockwise** so we set the default yaw value to.

`yaw = -90.0f;`

**DO NOT WORRY ABOUT THIS IMAGE THIS IS LEFT OVER FROM MY PREVIOUS EDITION OF THESE NOTES**

![[Pasted image 20250305112857.png]]

**DO NOT WORRY ABOUT THIS IMAGE THIS IS LEFT OVER FROM MY PREVIOUS EDITION OF THESE NOTES**

**Mouse Input**

The yaw and pitch values are obtained from the mouse (or controller/joystick) where horizontal mouse-movement affects yaw and vertical mouse-movement affects pitch. The idea is to store the last frame's (this of FPS in the gaming sense when you think of Frame) mouse positions and calculate in the current frame how much the mouse values have changed. The higher the horizontal or vertical difference, the more we update the pitch or yaw value and thus the more the camera should move. 

First we will tell GLFW that it should hide the cursor and **capture** it. Capturing a cursor means that, once the application has focus, the mouse cursor stays within the center of the window (unless the application loses focus or quits). We can do this with one simple configuration call. 

`glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);`

After this call, wherever we move the mouse it won't be visible and should not leave the window. This is perfect for an FPS camera system. 

To calculate the pitch and yaw values we need to tell GLFW to listen to mouse-movement events. We do this by creating a callback function with the following prototype.

`void mouse_callback(GLFWwindow* window, double xpos, double ypos);`

Here `xpos` and `ypos` represent the current mouse positions. As soon as we register the callback function with GLFW each time the mouse moves, the `mouse_callback` function is called. 

`glfwSetCursorPosCallBack(window, mouse_callback);`

When handling mouse input for a fly style camera there are several steps we have to take before we're able to fully calculate the camera's direction vector.

1. Calculate the mouse's offset since the last frame.
2. Add the offset values to the camera's yaw and pitch values.
3. Add some constraints to the minimum/maximum pitch values.
4. Calculate the direction vector.

The first step is to calculate the offset of the mouse since last frame. We first have to store the last mouse positions in the application, which we initialize to be in the center of the screen (screen size is  800 by 600)

`float lastX = 400, lastY = 300;`

Then in the mouse's callback function we calculate the offset movement between the last and current frame:

```
float xoffset = xpos - lastX;
float yoffset = lastY - ypos; // reversed since y-coordinates range from bottom to                                                                                 top
lastX = xpos;
lastY = ypos;

const float sensitivity = 0.1f;
xoffset *= sensitivity;
yoffset *= sensitivity;
```

Note we multiply the offset values by the sensitivity value. If we omit this multiplication the mouse movement would be too strong. 

Next we add the offset values to the to the globally declared pitch and yaw values. 

```
yaw += xoffset; 
pitch += yoffset;
```

add constraints to the camera so users won't be able to make weird camera movements. The pitch needs to be constrained in such a way that users won't be able to look higher than 89 degrees (at 90 degrees we get the LookAt flip). and also not below -89 degrees. This ensures the user will be able to look up to the sky or below to his feet but not further. The constraints work by replacing the Euler value with its constraint value whenever it breaches the constraint. 

```
if(pitch > 89.0f) 
pitch = 89.0f; 
if(pitch < -89.0f) 
pitch = -89.0f;
```

Lastly, we need to calculate the actual direction vector using the formula from the previous section.

```
glm::vec3 direction;
direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
direction.y = sin(glm::radians(pitch));
direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
cameraFront = glm::normalize(direction);
```

The computed direction vector then contains all the rotations calculated from the mouse's movement. Since the cameraFront vector is already included in glm's look at function, we're set to go. 

We also set a first mouse variable so that we stop getting that snapping movement whenever the window first receives focus of your mouse cursor. We solve this by declaring a global bool variable and then if it was the first time the mouse was on the screen. Set lastX and lastY to their respective coordinates with no subtraction. The code should look like this:

```
if (firstMouse) // initially set to true
{
    lastX = xpos;
    lastY = ypos;
    firstMouse = false;
}
```

Here is the mouse_callback function in full:

```
void mouse_callback(GLFWwindow* window, double xpos, double ypos)
{
    if (firstMouse)
    {
        lastX = xpos;
        lastY = ypos;
        firstMouse = false;
    }
  
    float xoffset = xpos - lastX;
    float yoffset = lastY - ypos; 
    lastX = xpos;
    lastY = ypos;

    float sensitivity = 0.1f;
    xoffset *= sensitivity;
    yoffset *= sensitivity;

    yaw   += xoffset;
    pitch += yoffset;

    if(pitch > 89.0f)
        pitch = 89.0f;
    if(pitch < -89.0f)
        pitch = -89.0f;

    glm::vec3 direction;
    direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
    direction.y = sin(glm::radians(pitch));
    direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
    cameraFront = glm::normalize(direction);
}  
```


Lastly we can create a zoom function and bind it to the scroll wheel. 

First we create a callback function called void scroll call back. This is what it looks like:


```
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
{
    fov -= (float)yoffset;
    if (fov < 1.0f)
        fov = 1.0f;
    if (fov > 45.0f)
        fov = 45.0f; 
}
```

When scrolling, the yoffset value tells us the amount we scrolled vertically. When the scroll_callback function is called we change the content of the globally declared fov variable. Since 45.0 is the default fov value we want to constrain the zoom level between 1.0 and 45.0.

We now have to upload the perspective with the global fov variable as the field of view.

`projection = glm::perspective(glm::radians(fov), 800.0f / 600.0f, 0.1f, 100.0f);`

Lastly, make sure you pre define the function above with its parameters and use the glfw callback function to activate it. 

`glfwSetScrollCallback(window, scroll_callback);`