
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

By telling OpenGL we're finished with the pointer operations via `glUnmapBuffer`, OpenGL knows you're done. By un-mapping, the pointer becomes invalid and the function returns `GL_TRUE` if OpenGL was able to map your data successfully to the buffer. 

Using `glMapBuffer` is useful for directly mapping data to a buffer, without first storing it in temporary memory. Think of directly reading data from file and copying it into the buffer's memory

**Batching Vertex Attributes**

Using `glVertexAttribPointer` we were able to specify the attribute layout of the vertex array buffer's content. Within the vertex array buffer we **interleaved** the attributes; that is, we placed the position, normal/or textures coordinates next to each other in memory for each vertex. Now that we know a bit more about buffers we can take a different approach. 

What we could do is batch all the vector data into large chunks per attribute type instead of interleaving them. Instead of an interleaved layout i.e. `123123123123`, we take a batched approach i.e. `111122223333`. 

When loading vertex data from file you generally retrieve an array of positions, an array of normals, or an array of texture coordinates. It may cost some effort to combine these arrays into one large array of interleaved data. Taking the batching approach is then an easier solution that we can easily implement using `glBufferSubData`.

```
float positions[] = {...};
float normals[] = {...};
float tex[] = {...};
// fill buffer
glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(positions), &positions);
glBufferSubData(GL_ARRAY_BUFFER, sizeof(positions), sizeof(normals), &normals);
glBufferSubData(GL_ARRAY_BUFFER, sizeof(positions) + sizeof(normals), sizeof(tex), &tex);
```

This way we can directly transfer the attribute arrays as a whole into the buffer without first having to process them. We could have also combined them in one large array and fill the buffer right away using `glBufferData`, but using `glBufferSubData`