
As soon as we compute the final pixel colors of the scene we will have to display them on a monitor. In the old days of digital imaging most monitors were cathode-ray tube (CRT) monitors. These monitors had the physical property that twice the input voltage did not result in twice the amount of brightness. Doubling the input voltage resulted in a brightness equal to an exponential relationship of roughly 2.2 known as the **gamma** of a monitor. This happens to (coincidently) match how humans beings measure brightness as brightness is also displayed with a similar (inverse) power relationship. To better understand what this all means take a look at the following image. 

![[Pasted image 20251020113628.png]]

The top line looks like the correct brightness scale to the human eye, doubling the brightness (from 0.1 to 0.2 for example) does indeed look like its twice as bright with nice consistent differences. However, when we're talking about the physical brightness of light e.g. amount of photons leaving a light source, the bottom scale actually displays the correct brightness. At the bottom scale, doubling the brightness returns the correct physical brightness, but since our eyes perceive brightness differently (more susceptible to changes in dark colors) it looks weird. 

Because the human eyes prefer to see brightness colors according to the top scale, monitors (still today) use a power relationship for displaying output colors so that the original physical brightness colors are mapped to the non-linear brightness colors in the top scale. 

This non-linear mapping of monitors does output more pleasing brightness results for our eyes, but when it comes to rendering graphics there is one issue: all the color and brightness options we configure in our applications are based on what we perceive from the monitor and thus all the options are actually non-linear brightness/color options. Take a look at the graph below. 

![[Pasted image 20251020122921.png]]

The dotted line represents color/light values in linear space and the solid line represents the color space that monitors display. If we double a color in linear space, its result is indeed double the value. For instance, take a light's color vector (0.5, 0.0, 0.0) which represents a slightly dark red light. If we would double this light in linear space it would become (1.0, 0.0, 0.0) as you can see in the graph. However, the original color gets displayed on the monitor as (0.218, 0.0, 0.0) as you can see from the graph. Here's where the issues start to rise: once we double the dark-red light in linear space, it actually becomes more than 4.5 times as bright on the monitor!

Up until this section of notes we have assumed we are working in linear space, but we've actually been working in the monitors output space so all colors and lighting variables we configured weren't physically correct, but merely looked (sort of) right on our monitor. For this reason, we (and artists) generally set lighting values way brighter than they would be (since the monitor darkens them) which as a result makes most linear-space calculations incorrect. Note that the monitor (CRT) and linear graph both start and end at the same position; it is the intermediate values that are darkened by the display. 

Because colors are are configured based on the display's output, all intermediate (lighting) calculations in linear-space are physically incorrect. This becomes more obvious as more advanced lighting algorithms are in place, as you can see in the image below. 

![[Pasted image 20251020131735.png]]

You can see that with gamma correction, the (updated) color values work more nicely together and darker areas show more details. Overall, a better image quality with a few small modifications. 

Without properly correcting this monitor gamma, the lighting looks wrong and artists will have a hard time getting realistic and good-looking results. The solution is to apply **gamma correction**.

**Gamma Correction**

The idea of gamma correction is to apply the inverse of the monitor's gamma to the final output color before displaying to the monitor. Looking back at the gamma curve from earlier in the notes we see another dashed line that is the inverse of the monitor's gamma curve. We multiply each of the linear output colors by this inverse gamma curve (making them brighter) and as soon as the colors are displayed on the monitor, the monitor's gamma curve is applied and the resulting colors become linear. We effectively brighten the intermediate colors so that as soon as the monitor darkens them, it balances all out.

Let's give another example. Say we again have the dark-red color (0.5, 0.0, 0.0). Before displaying this color to the monitor we first apply the gamma correction curve to the color value. Linear colors displayed by a monitor are roughly scaled to a power of 2.2 so the inverse requires scaling colors by a power of $1/2.2$. 
The gamma