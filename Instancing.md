
Say you have a scene where you're drawing a lot of models where most of these models contain the same set of vertex data, but with different world transformations. Think of a scene filled with grass leaves: each grass leave is a small model that consists of only a few triangles. You'll probably want to draw quite a few of them and your scene may end up with thousands or maybe tens of thousands of grass leaves that you need to render each frame. Because each leaf is only a few triangles, the leaf is rendered almost instantly. However, the thousands of render calls you'll have to make will drastically reduce performance. 

If we were to actually render such a large amount of objects it will look a bit like this in code. 

```
for (unsigned int i = 0; i < amount_of_models_to_draw; i++)
{
	DoSomePreperations(); // bind VAO, bind textures, set uniforms etc.
	glDrawArrays(GL_TRIANGLES, 0, amount_of_vertices);
}
```

When drawing many **instances** of your model like this you'll quickly reach a performance bottle neck because of the many draw calls. Compared to rendering the actual vertices, telling the GPU to render 