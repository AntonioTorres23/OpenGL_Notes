
Throughout most chapters we've been extensively using buffers in OpenGL to store data on the GPU. We'll briefly discuss a few alternative approaches to managing buffers. 

A buffer in OpenGL, at its core, an object that manages a certain piece of GPU memory and nothing more. We give meaning to a buffer when binding it to a specific buffer target. A buffer is only a vertex buffer when we bind it to `GL_ARRAY_BUFFER`, but we could easily bind it to `GL_ELEMENT_ARRAY_BUFFER`. OpenGL internally stores a reference to the buffer per target and, based on the target, processes the buffer differently. 

So far we've been filling the buffer's memory by calling `glBufferData`, which allocates a piece of GPU memory and adds data into this memory. If we were to pass null as its data argument, the function would only allocate memory and not fill it. This is useful if we first want to *reserve* a specific amount of memory and later come back to this buffer. 

Instead of filling the entire buffer with one function call we can also fill specific regions of the buffer by calling `glBufferSubData`. This function expects a buffer target, an offset, the size of the data, and the actual data as its arguments. What's new with this function is that we can now give an offset that specifies from *where* we want to fill the buffer. This allows us to insert/update only certain parts of the buffers memory. Do note that the buffer should have enough allocated memory so a call to `glBufferData` is necessary before calling `glBufferSubData` is necessary before calling `glBufferSubData` on the buffer. 

```
// Range: [24, 24 + sizeof(data)]
glBufferSubData(GL_ARRAY_BUFFER, 24, sizeof(data), &data);
```

Yet another method for getting data into a buffer is to ask for a pointer to the buffer's memory and directly copy the data in memory yourself. By calling `glMapBuffer` OpenGL returns a pointer to the currently bound buffer's memory for us to operate on. 

```
float data[] = 
{
0.5, 1.0f, -0.35f
};

glBindBuffer(GL_ARRAY_BUFFER, buffer);
// get pointer
// 2nd argument can be GL_WRITE_ONLY, GL_READ_ONLY, or GL_READ_WRITE
// generates a void pointer to the memory content within the binded buffer 
void *ptr = glMapBuffer(GL_ARRAY_BUFFER, GL_WRITE_ONLY);
// built-in C++ function that copies memory address of a variable
memcpy(ptr, data, sizeof(data));
// make sure to tell OpenGL we're done with the pointer
glUnmapBuffer(GL_ARRAY_BUFER);
```

By telling OpenGLK 
