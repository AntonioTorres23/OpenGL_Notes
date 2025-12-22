
At some stage of your graphics adventures you'll want to draw text in OpenGL. Contrary to what you may expect, getting a simple string to render on screen is all but easy with a low-level API like OpenGL. If you don't care about rendering more than 128 different same-sized, then it's probably not too difficult. Things are getting difficult as soon as each character has a different width, height, and margin. Based on where you live, you may also need more than 128 characters, and what if you want to express special symbols for like mathematical expressions or sheet music symbols, and what about rendering text from top to bottom? Once you think about all these complicated matters of text, it wouldn't surprise you that this probably doesn't belong in a low-level API like OpenGL. 

Since there is no support for text capabilities within OpenGL, it is up to us to define a system for rendering text to the screen. There are no graphical primitives for text characters, we have to get creative. Some example techniques are: drawing letter shapes via `GL_LINES`, create 3D meshes of letter, or render character textures to 2D quads in a 3D environment. 

Most developers choose to render charact textures onto quads. Rendering textured quads by itself shouldn't be too difficult, but getting the relevant character(s) onto a texture could prove challenging. In these notes we'll explore several methods and implement a more advanced, but flexible technique for rendering text using the FreeType library. 

**Classical Text Rendering: Bitmap Fonts**

In the early days, rendering text involved selecting a font (or creating one yourself) you'd like for your application and extracting all relevant characters out of this font to place them within a single large texture. Such a texture, that we call a **bitmap font**, contains all character symbols we want to use in predefined regions of the texture. These character symbols of the font are known as **glyphs**. Each glyph has a specific region of texture coordinates associated with them. Whenever you want to render a character, you  select the corresponding glyph by rending this section of the bitmap font to a 2D quad. 

![[Pasted image 20251222140117.png]]

Here you can see how we would render the text `OpenGL` by taking a bitmap font and sampling the corresponding glyphs from the texture (carefully choosing the texture coordinates) that we render on top of several quads. By enabling [blending](https://learnopengl.com/Advanced-OpenGL/Blending) and keeping the background transparent, we will end up with just a string of characters to the screen. This particular bitmap font was generated using Codehead's Bitmap [Font Generator](http://www.codehead.co.uk/cbfg/).

This approach has several advantages and disadvantages. It is relatively easy to implement and because bitmap fonts are pre-rasterized, they're quite efficient. However, it is not particularly flexible. When you want to use a different font, you need to recompile a complete new bitmap font and the system is limited to a single resolution; zooming will quickly show pixelated edges. Furthermore, it is limited to a small character set, so Extended or Unicode characters are often out of the question. 

This approach was quite popular back in the day (and still is) since it is fast and works on any platform, but as of today more flexible approaches exist. One of these approaches is loading TrueType fonts using the FreeType library. 

**Modern Text Rendering: FreeType**

FreeType is a software development library that is able to load fonts, render them to bitmaps, and provide support for several font-related operations. It is a popular library used by Mac OS X, Java, PlayStation, Linux, and Android to name a few. What makes FreeType particularly attractive is that it is able to load TrueType fonts. 

A TrueType font is a collection of character glyphs not defined by pixels or any other non-scalable solution, but by mathematical equations (combinations of splines). Similar to vector images, the rasterized font images can be procedurally generated based on the preferred font height you'd like to obtain them in. By using TrueType fonts you can easily render character glyphs of various sizes without any loss of quality. 

FreeType can be downloaded from their [website](http://www.freetype.org/). You can choose to compile the library yourself or use one of their precompiled libraries if your target platform is listed. Be sure to link to `freetype.lib` and make sure your compiler knows where to find the header files. 

Then include the appropriate headers. 

```
#include <ft2build.h>
#include FT_FREETYPE_H
```

Due to how FreeType is developed (at least at the time of this writing), you cannot put their header files in a new directory 
