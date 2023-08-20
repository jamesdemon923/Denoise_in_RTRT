# Denoise in real time ray tracing

## Setup

**Operating & compiling environment**:

* Visual studio 2022 (Windows 11)

**Scripts:**

* Use **build.bat**/**build.sh** to build the project
* **image2video** used to transfer the output from pictures to videos (Need [ffmpeg](https://ffmpeg.org/))

## Joint Bilateral Filter

For denoising, we can filter out the noisy signal by various filters. 

The simplest one is **Gaussian filter**. For the final color of a target pixel, Gaussian filter will consider the color contribution from all adjacent pixels, and the weight of the contribution decreases with increasing distance from the target pixel, which ultimately achieves the purpose of removing the high frequency signal. However, this kind of filtering will also <u>blur the boundary of the object</u>, and we usually want to remove only the noise and maintain the object boundary sharp.

**Bilateral filter** is a good solution for this problem. Since there are often drastic changes about color in two sides of the object boundary, the bilateral filter takes into account the difference (distance) between colors, i.e., if the larger the difference between the color of the two pixels, the smaller the contribution in order to retain the edge information. 

**Joint bilateral filter**, on the other hand, takes into account more reference information, such as depth, normal, and world coordinates to better guide the filtering operation.

In the following picture, **A and B need considering depth; B and C need considering normal; D and E need considering color.**

<div align=center>
    <img src="images\Theory\Joint Bilateral Filtering.png" width="400" />
</div> 


### Filter kernel:

<div align=center>
    <img src="images\Equation\Filter Kernel.png" width="600" />
</div> 

<div align=center>
    <img src="images\Equation\Dnormal.png" width="400" />
</div> 

<div align=center>
    <img src="images\Equation\Dplane.png" width="400" />
</div> 

In the above equations, $\widetilde{C}$ is the noisy input image, $D_{normal}$ is the angle between the normals of two points (**for normal information**), $D_{plane}$ provides a better metric than just simply calculating the difference between two depths (**for depth information**).

### Temporal Accumulation:

#### Motion vector

The projection equation:

<div align=center>
    <img src="images\Equation\Projection.png" width="400" />
</div> 

This is the advantage of graphics: **mastering the matrices of the entire pipeline for calculating conveniently.**

#### Accumulation:

<div align=center>
    <img src="images\Equation\Temporal accumulation.png" width="300" />
</div> 

For clamping, it is first necessary to compute the mean $\mu$ and variance $\sigma$ of $\bar{C}^{(i)}$ in a 7x7 neighborhood, and then to clamp the color of the previous frame $C^{i-1}$ into the range $(\mu-k\sigma,\mu+k\sigma)$.

### A-Trous wavelet:

<div align=center>
    <img src="images\Theory\A-Trous Wavelet.png" width="500" />
</div> 

## Pipeline

| Step |                          Operation                           |
| :--: | :----------------------------------------------------------: |
|  1   |                    Denoise for per frame                     |
|  2   | Project the last frame into the present one to implement the temporal accumulation |
|  3   |       Accelerate the denoise based on A-Trous wavelet        |

## Result

<table>
    <tr>
        <th colspan="1">Before denoise</th>
        <th colspan="1">After denoise</th>
    </tr>
    <tr>
        <td ><center><img src="images/Result/Before denoise.jpg"></center></td>
        <td ><center><img src="images/Result/After denoise.jpg"></center></td>
    </tr>

<table>
    <tr>
        <th colspan="2">Result of cornell box</th>
    </tr>
    <tr>
        <td ><center><img src="images/Result/box-input.gif">Input</center></td>
        <td ><center><img src="images/Result/box-After denoise.gif">After denoise for per frame</center></td>
    <tr>
    <tr>	
        <td ><center><img src="images/Result/box-After TemAccumulation.gif">After temporal accumulation</center></td>
        <td ><center><img src="images/Result/box-with A-Trous.gif">Accelerated by A-Trous</center></td>
    </tr>

<table>
    <tr>
        <th colspan="2">Result of pink house (see the original results in images/result/.mp4 file)</th>
    </tr>
    <tr>
        <td ><center><img src="images/Result/pinkroom-input.gif">Input</center></td>
        <td ><center><img src="images/Result/pinkroom-Only TemAccumulation without filter.gif">Only temporal accumulation without filter</center></td>
    <tr>
    <tr>	
        <td ><center><img src="images/Result/pinkroom-After TemAccumulation.gif">After temporal accumulation</center></td>
        <td ><center><img src="images/Result/pinkroom-with A-Trous.gif">Accelerated by A-Trous</center></td>
    </tr>

|                 | Cornell Box | Pink House |
| :-------------: | :---------: | :--------: |
| Without A-Trous |    38''     |   9'22''   |
|  With A-Trous   |     8''     |   1'42''   |

