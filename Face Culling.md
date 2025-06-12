Try mentally visualizing a 3D cube and count the maximum number of faces you'll be able to see from any direction. If your imagination isn't too creative you probably ended up with a maximum number of 3. You can view a cube from any position and or direction, but you would never be able to see more than 3 faces. So why would we waste the effort of drawing those other 3 faces that we can't even see. If we could discard those in some way we would save more than 50% of this cube's total fragment shader runs. 

We say more than 50% instead of 50%, because from certain angles only 2 or even 1 face could be visible. In that case we would save more than 50%.

This is a really great idea, but there's one problem we need to solve. How do we know if a face of an object is not visible from the viewer's POV? If we image any closed shape, each of its faces has two sides. Each side would either face the user or show its back to the user. What if we could only render the faces that are facing the viewer.

This is exactly what face culling does. OpenGL checks all the faces that are front facing towards the viewer and renders those wile discarding all the faces that are back facing, saving us a lot of fragment shader calls. We do need to tell OpenGL which of the faces we use are actually the front faces and which faces are the back faces. OpenGL uses a clever trick for this by analyzing the winding order of the vertex data. 

**Winding Order**

When we define a set of triangle vertices we're defining them in a certain winding order that is either clockwise or counter-clockwise. Each triangle consists of 3 vertices and we specify those 3 vertices in a winding order as seen from the center of the triangle. 

![[Pasted image 20250603162333.png]]
As you can see in the image we first define vertex 1 and from there we can choose whether the next vertex is 2 or 3. This choice defines the winding order of this triangle. The following code illustrates this. 

`float vertices[] = {`
	`// clockwise`
	`vertices[0], // vertex 1`
	`vertices[1], // vertex 2`
	`vertices[2], // vertex 3`
	`// counter-clockwise`
	`vertices[0], // vertex 1`
	`vertices[2], // vertex 3`
	`vertices[1]1 // vertex 2`
`};`

Each set of 3 vertices that from a triangle primitive thus contain a winding order. OpenGL uses this information when rendering your primitives to determine if a triangle is front-facing or a back-facing triangle. By default, triangles defined with counter-clockwise vertices are processed as front-facing triangles. 

When defining your vertex order you visualize the corresponding triangle as if it was facing you, so each triangle that you're specifying should be counter-clockwise as if you're directly facing that triangle. The cool thing about specifying all your vertices like this is that the actual winding order is calculated at the rasterization stage, so when the vertex shader has already run. The vertices are then seen as from the vertices are then seen as from the viewer's point of view. 

All the triangle vertices that the viewer is then facing are indeed in the correct winding order as we specified them, but the vertices of the triangles at the other side of the cube are now rendered in such a way that their winding order now becomes reversed. The result is that the triangle's we're facing are seen as front-facing triangles and the triangles at the back are seen as back-facing triangles. The following image shows this effect.

![[Pasted image 20250603165126.png]]

In the vertex data we defined both triangles in counter-clockwise order (the front and back triangle as 1,2,3). However, from the viewer's direction the back triangle is rendered clockwise if we draw it in the order of 1,2,3 from the viewer's current point of view. Even though we specified the back triangle in counter-clockwise order, it is now rendered in clockwise order. This is exactly what we want to cull (discard) non-visible faces. 

**Face Culling**

At the beginning we said that OpenGL is able to discard triangle primitives if they're rendered as back-facing triangles. Now that we know how to set the winding order of the vertices we can start using OpenGL's face culling option which is disabled by default.

The cube vertex data we used in the previous chapters wasn't defined with the counter-clockwise winding order in mind, so I updated the vertex data to reflect a counter-clockwise winding order which you can copy from [here](https://learnopengl.com/code_viewer.php?code=advanced/faceculling_vertexdata). It's a good practice to try and visualize that these vertices are indeed all defined in a counter-clockwise order for each triangle.

To enable face culling we only have to enable OpenGL's GL_CULL_FACE option.

`glEnable(GL_CULL_FACE);`

From this point on, all the faces that are not front-faces are discarded (try flying inside the cube to see that all inner faces are indeed discarded). Currently we save over 50% of performance on rendering fragments if OpenGL decides to render the back faces first (otherwise depth testing would've discarded them already). Do note that this only really works with closed shapes like a cube. We do have to disable face culling again when we draw the grass leaves from the previous lesson, since their front and back face should be visible. 

OpenGL allows us to change the type of face we want to cull as well. What if we want to cull front faces and not the back faces? We can define this behavior with `glCullFace`.

`glCullFace(GL_FRONT);`

The `glCullFace` function has three possible options:
- `GL_BACK`: Culls only the back faces.
- `GL_FRONT`: Culls only the front faces. 
- `GL_FRONT_AND_BACK`: Culls both front and back faces. 

The initial values of `glCullFace` is `GL_BACK`. We can also tell OpenGL we'd rather prefer clockwise faces as the front-faces instead of counter-clockwise faces via `glFrontFace`.

`glFrontFace(GL_CCW);`

The default value is `GL_CCW` that stands for counter-clockwise ordering with the other option being `GL_CW` which stands for clockwise ordering. 

As a simple test we could reverse the winding order by telling OpenGL that the front-faces are now determined by a clockwise ordering instead of a counter-clockwise ordering. 

`glEnable(GL_CULL_FACE);`
`glCullFace(GL_BACK);`
`glFrontFace(GL_CW);`

The result is that only the back faces are rendered. 

![[Pasted image 20250604165359.png]]

Note that you can create the same effect by culling front faces with the default counter-clockwise winding order.

`glEnable(GL_CULL_FACE);`
`glCullFace(GL_FRONT);`

As you can see, face culling is a great tool for increasing performance of your OpenGL applications with minimal effort. Especially as all 3d applications export models with consistent winding orders (CCW by default). You do have to keep track of the objects that will actually benefit from face culling and which objects shouldn't be culled at all. 