
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

Next we have a camera front position which is where the camera is pointing. 

`glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);`

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
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight); // cams up position                                                                   vector
```

Lastly, we have a camera up position for the y axis to set vertical height on the camera. 

`glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);`

To get a look at function we use these three vectors and add them within GLFW's look at function:

`view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);`

The view variable holds the look at function which takes 3 parameters, the position of the camera, the target at which the camera wants to point at. As well as a vector that creates the right vector

Here is the definition that are for the lookat function parameters that LearnOpenGl provides. 

1st position: Specifies the position of the camera in word coordinates.

2nd target: Specifies the target direction/position the camera should look at. 

3rd up: specifies a vector pointing in the positive y direction used to create the right vector. This is usually set at (0.0, 1.0, 0.0).

This is all preformed within the camera header file. 

Why we add the position and front values for the y coordinate so however we move, the camera keeps looking at the target direction. 


When the camera object is created, it has predefined values that are used for the lookat function. This is stored within a function called GetViewMatrix within the camera header file. 

Within the main file, this is stored within the view variable and called via the camera object that was created earlier in the function. 

When the camera object is created with the header file, the predefined vector variables get data entered into them. 

Then to bind them to a key we us the pre-defined open GL variables and put them into the process input function. Then if we move up or back, we subtract or add the position front vector to the camera position. If we move from side to side, we subtract or add the normalized cross product of camera front and camera up to the position camera.  

You need to multiply the Front camera by the Front and Up vectors to get the Right vector which allows you to move along the x coordinate path. 

Delta time is a variable that stores the time it took to render the last frame. To get that you subtract the current frame from the last frame.

```
float deltaTime = 0.0f; // Time between current frame and last frame float lastFrame = 0.0f; // Time of last frame

float currentFrame = glfwGetTime(); 
deltaTime = currentFrame - lastFrame; 
lastFrame = currentFrame;

```

Then you multiply this in a variable by the speed you want your camera at to create a buffer if a frame took longer than average which will slow the camera down.

```
  
void processInput(GLFWwindow *window) { float cameraSpeed = 2.5f * deltaTime; [...] }
```


This covers the part of controlling the camera to move in 3d but the other half after this sentence is how to look around in a 3d environment using an input such as mouse. 

**Pitch** is how up or down we are looking at an object in 3d and is controlled by the y coordinate. 

**Yaw** is how much we are looking left or right at an object in 3d and it is determined by the x coordinate. 

So using trigonometry, you can use the cos and sin to get the position of x and y. 


*"Let's start with a bit of a refresher and check the general right triangle case (with one side at a 90 degree angle):*"



 ![[Pasted image 20250304162505.png]]
`

*"If we define the hypotenuse to be of length `1` we know from trigonometry (soh cah toa) that the adjacant side's length is cos x/h=cos x/1=cos xcos⁡ x/h=cos⁡ x/1=cos⁡ x and that the opposing side's length is sin y/h=sin y/1=sin ysin⁡ y/h=sin⁡ y/1=sin⁡ y. This gives us some general formulas for retrieving the length in both the `x` and `y` sides on right triangles, depending on the given angle. Let's use this to calculate the components of the direction vector.*

*Let's imagine this same triangle, but now looking at it from a top perspective with the adjacent and opposite sides being parallel to the scene's x and z axis (as if looking down the y-axis)."*

![[Pasted image 20250304162642.png]]


*"If we visualize the yaw angle to be the counter-clockwise angle starting from the `x` side we can see that the length of the `x` side relates to `cos(yaw)`. And similarly how the length of the `z` side relates to `sin(yaw)`."*

"*If we take this knowledge and a given `yaw` value we can use it to create a camera direction vector:*"

```

glm::vec3 direction;
direction.x = cos(glm::radians(yaw)); // Note that we convert the angle to radians first
direction.z = sin(glm::radians(yaw));
```

This is how to get the Yaw of an object in 3d but we need to get the Pitch as well. 

This is accomplished similarly to Yaw where x and z are meeting with y to get an angle. 


![[Pasted image 20250305111404.png]]

As before we use sin to get the pitch of y. 

However from the pitch triangle we can also see the xz sides are influenced by cos so we need to make sure this is also part of the direction vector. 

With this included, we get the final direction vector as translated from yaw and pitch euler angles. 



```
direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch)); direction.y = sin(glm::radians(pitch)); direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
```

We've set up the scene world so everything is positioned in the direction of the negative z axis. However, if we look at the x and z yaw triangle we see that a 0 of 0 results in the camera's direction vector to point towards the positive x-axis. 

To make sure the camera points towards the negative z-axis by default, we can give the yaw a default value of a 90 degree clockwise rotation. Positive degrees rotate counter-clockwise so we set the default yaw value to:

`yaw = -90.0f;`


![[Pasted image 20250305112857.png]]


To use these values in a game design sense, we are going to capture input from the mouse's movement. Where horizontal movement affects the yaw and vertical movement affects the pitch. The idea is to store the last frame's mouse positions and calculate in the current frame how much the mouse values has changed. The higher the horizontal or vertical difference, the more we update the pitch or yaw value and thus the more the camera should move. 


First tell GLFW that it should hide the cursor and capture it. Capturing a cursor means that once the application has focus, the mouse cursor stays within the center of the window (unless the application losses focus or quits)

`glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);`

After this, to calculate the pitch and yaw values we need to tell GLFW to listen to mouse-movement events. We do this by creating a callback function with the following prototype:

`void mouse_callback(GLFWwindow* window, double xpos, double ypos);`

Here xpos and ypos represent the current mouse positions. As soon as we register the callback function with GLFW each time the mouse moves the mouse callback function is called. 

`glfwSetCursorPosCallback(window, mouse_callback);`


To handle mouse input for a fly style cam, we need to do several steps before we can fully calculate the camera's direction vector. 

1. Calculate the mouse's offset since the last frame.
2. Add some offset values to the camera's yaw and pitch values.
3. Add some constraints to the minimum/maximum pitch values.
4. Calculate the direction vector.

The first step is to calculate the offset of the mouse since the last frame. We first have to store the last mouse positions in the application, which we initialize to be in the center of the screen (screen is 800x600). 

`float lastX = 400, lastY = 300;`

Then in the mouse's callback function we calculate the offset movement between the last and current frame:

```

float xoffset = xpos - lastX;
float yoffset = lastY - ypos; // reversed since y-coordinates range from bottom to top
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