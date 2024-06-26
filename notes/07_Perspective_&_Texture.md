# 07 Perspective & Texture

## 7.1 Perspective Projection

**clipping**

In real-time graphics pipeline, “clipping” is the process of eliminating triangles that aren’t visible to the camera

![1544526666735](assets/1544526666735.jpg)

**Near/Far Clipping**

important for dealing with fnite precision of depth buffer / limitations on storing depth as foating point values 

foating point has more “resolution” near zero 

> The narrower the near/far, the higher the accuracy.

**Matrix for Perspective Transform**
$$
\begin{bmatrix}
\frac{2n}{r-l} & 0 & \frac{r+l}{r-l} & 0 \\
0 & \frac{2n}{t-b} & \frac{t+b}{t-b} & 0 \\
0 & 0 & \frac{-(f+n)}{f-n} & \frac{-2fn}{f-n}\\
0 & 0 & -1 & 0 \\
\end{bmatrix}
$$
left (l), right (r), top (t), bottom (b), near (n), far (f) 

**Barycentric Coordinates**

![1544529768539](assets/1544529768539.jpg)
$$
\phi_i(x)=\frac{\text{area}(x,x_j,x_k)}{\text{area}(x_i,x_j,x_k)}
$$

> $Attr(x)=\phi_i(x)Attr(x_i)+\phi_j(x)Attr(x_j)+\phi_k(x)Attr(x_k)$

**Perspective-incorrect interpolation**

Due to perspective projection (homogeneous divide), barycentric interpolation of values on a triangle with different depths is not an affine function of screen XY coordinates. 

Attribute values must be interpolated linearly in 3D object space. 

![1544530020221](assets/1544530020221.jpg)

**Perspective Correct Interpolation**

- To interpolate some attribute ɸ…

- Compute depth z at each vertex
- Evaluate Z := 1/z and P := ɸ/z at each vertex
- Interpolate Z and P using standard (2D) barycentric coords
- At each fragment, divide interpolated P by interpolated Z to get fnal value 

> $$
> \begin{aligned}
> Z_i&=1/z_i\\
> P_i&=Attr(x_i)/z_i\\
> Attr(x)&=\frac{\phi_i(x)P_i+\phi_j(x)P_j+\phi_k(x)P_k}{\phi_i(x)
> Z_i+\phi_j(x)Z_j+\phi_k(x)Z_k}
> \end{aligned}
> $$
>

## 7.2 Texture Mapping

**101**

Basic algorithm for mapping texture to surface: 

- Interpolate U and V coordinates across triangle 
- For each fragment 
  - Sample (evaluate) texture at (U,V) 
  - Set color of fragment to sampled texture value 

sadly not this easy in general! 

**Filtering textures**

![1544530450514](assets/1544530450514.jpg)

**Mipmap**

![1544530494491](assets/1544530494491.jpg)

preflter texture data to remove high frequencies 

Texels at higher levels store integral of the texture function over a region of texture space (downsampled images) 

Texels at higher levels represent ==low-pass fltered== version of original texture signal 

**Compute Mipmap Level**

![1544530598797](assets/1544530598797.jpg)

**Bilinear filtering**

![1545460392164](assets/1545460392164.jpg)
$$
\begin{aligned}
P
&=(1-t)((1-s)P_{00}+sP_{01})+t((1-s)P_{10}+sP_{11})\\
&=(1-t)(1-s)P_{00}+(1-t)sP_{01}+t(1-s)P_{10}+stP_{11}
\end{aligned}
$$
**Tri-linear filtering**

Linear interpolation between two levels, bilinear interpolation within the level, resulting in trilinear interpolation

![1544530745171](assets/1544530745171.jpg)

![1544530772836](assets/1544530772836.jpg)

> There is a key point that is not mentioned here, which is how to select the four surrounding points when the location of the interpolation point is known.
>
> You can tell just by looking at the picture, just look at which corner of the pixel it falls on
>
> In the picture on the left, the interpolation point is at the upper left corner of the pixel center, so select the four pixels in the upper left corner
>
> In the picture on the right, the interpolation point is in the lower right corner of the pixel center, so select the four pixels in the lower right corner
>
> The general rule is that the interpolation point is within the center points of the four pixels

**anisotropic filter**

The above image is sampled using a square. However, the pixel may represent a long strip, so using a square would be too blurry.

![1544530930695](assets/1544530930695.jpg)

**Summary: texture fltering using the mip map**

- Small storage overhead (33%)

  - Mipmap is 4/3 the size of original texture image

- For each isotropically-fltered sampling operation

  - Constant fltering cost (independent of mip map level)

    > Because it has been calculated in advance (pre-filter)

  - Constant number of texels accessed (independent of mip map level)

    > Trilinear interpolation samples 8 pixels

- Combat aliasing with prefltering, rather than supersampling

- Bilinear/trilinear fltering is isotropic and thus will “overblur” to avoid aliasing

- Anisotropic texture fltering provides higher image quality at higher compute and memory bandwidth cost (in practice: multiple mip map samples) 

**"Real" Texture Sampling**

1. Compute u and v from screen sample x,y (via evaluation of attribute equations)
2. Compute du/dx, du/dy, dv/dx, dv/dy differentials from screen-adjacent samples.
3. Compute mip map level d
4. Convert normalized [0,1] texture coordinate (u,v) to texture coordinates U,V in [W,H]
5. Compute required texels in window of flter
6. Load required texels (need eight texels for trilinear)
7. Perform tri-linear interpolation according to (U, V, d) 

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({ tex2jax: {inlineMath: [['$', '$']]}, messageStyle: "none" });
</script>