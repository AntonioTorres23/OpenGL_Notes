
Brightness and color values, by default, are clamped between $0.0$ and $1.0$ when stored into a framebuffer. This, at first seemingly innocent, statement caused us to always specify light and color values somewhere in this range, trying to make them fit into the scene. This works ok and gives decent results, but what happens if we walk in a really bright area with multiple bright light sources that as a total sum exceed 1.0? The answer is that all fragments that have a brightness or color sum over $1.0$ gets clamped to $1.0$, which isn't pretty to look at. 

![[Pasted image 20251114164910.png]]

Due to a large number of fragments' color value getting clamped to $1.0$, each of the bright fragments have the exact same white color value in large regions, losing a significant amount of detail and giving it a fake look. 

A solution to this problem would be to reduce the strength of the light sources and ensure no area of fragments in your scene ends up brighter than ; this is not a good solution as this forces you to use unrealistic lighting parameters. A better approach is to allow color values to temporarily exceed $1.0$ and transform them back to the original range of $0.0$ and $1.0$ as a final step, but without losing detail. 

Monitors (non-HDR) are limited to display colors in the range of $0.0$ and $1.0$, but there is no such limitation in lighting equations. By allowing fragment colors to exceed $1.0$ we have a much higher range of color values available to work in known as **High Dynamic Range** (HDR). With high dynamic range, bright things can be really bright, dark things can be really dark, and details can be seen in both. 

High dynamic range was originally only used for photography where a photographer takes multiple pictures of the same scene with varying exposure levels, capturing a large range of color values. Combining these forms a HDR image where a range of details are visible based on the combined exposure levels, or a specific exposure it is viewed with. For instance, the following image (credits to Colin Smith) shows a lot of detail at brightly lit regions with a low exposure (look at the window), but these details are gone with a high exposure. However, a high exposure now reveals a great amount of detail at darker regions that weren't previously visible. 

![[Pasted image 20251117150757.png]]

This is also very similar to how the human eye works and the basis of high dynamic range rendering. When there is little light, the human eye adapts itself so the darker parts become more visible and similarly for bright areas. It's like the human eye has an automatic exposure slider on the scene's brightness.

High dynamic range rendering works a bit like that. We allow for a much larger range of color values to render to, collection a large range of dark and bright details of a scene, and at the end we transform all HDR values back to **low dynamic range** (LDR) of $[0.0, 1.0]$. This process of converting HRD values to LDR values is called **tone mapping** and a large collection of tone mapping algorithms exist that aim to preserve most HDR details during the conversion process. These tone mapping algorithms often involve an exposure parameter that selectively favors dark or bright regions. 

When it comes to real-time rendering, high dynamic range allows us to not only exceed the LDR range of $[0.0, 1.0]$ 
 








































