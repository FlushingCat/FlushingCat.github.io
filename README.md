<p align="center">
    <picture>
    <img src="https://github.com/FlushingCat/LandMark_Media_Content/blob/main/logo.png?raw=true" width="600">
    </picture>
</p>

<p align="center"> <font size="4"> 🌏NeRF the globe if you want </font> </p>

# Introduction
There are many one-of-a-kind highlights in the LandMark:

- Large-scale, high-precision Reconstrution
    - For the first time, efficient training of 100 square kilometers of city-scale NeRF was realized, and the rendering resolution reached 4K with model parameters over 200 billions.
- Multiple feature extentions
    - Rich capabilities beyond reconstruction, including ajusting urban layout such as removing or adding a buildings, and ajusting appearance style such as lighting changes related to seasons.
- Training, rendering integrated system
    - City-scale NeRF system covering algorithms, operators, computing systems, which provides a solid foundation for the training, rendering and application of real-world 3D large models.

The LandMark supports multiple types of parrllel strategies. With the strategies, we achieve huge NeRF speedups and make the NeRF truly applicable to city-scale reconstruction. These parallel strategies mainly include the following types:

- [Branch Parrallel](#branch-parrallel)
- [Plane Parrallel](#plane-parrallel)
- [Channel Parrallel](#plane-parrallel)

Here we present how our parrallel methods work. Basic knowledges of NeRFs are needed for better understanding.
# Branch Parrallel

<p align="center">
<picture>
    <img src="https://github.com/FlushingCat/LandMark_Media_Content/blob/main/multi-branch%20model.png?raw=true" width="600">
</picture>
</p>

The scene area is divided into multiple scene blocks. Correspondingly, the model changes from having a single large grid to multiple sub-grids. with each sub-grid representing a scene block. 

All sub-grids share the same MLPs to decode features and ouput rgbσ. All the sub-grid and the shared MLPs consitiute a complete multi-branch large model. 

When a point sampled along the ray is fed into the multi-branch model, it is first assigned to the corresponding sub-grid based on its coordinate, and then inferred with this branch.

<p align="center">
<picture>
    <img src="https://github.com/FlushingCat/LandMark_Media_Content/blob/main/Branch.png?raw=true" width="500">
</picture>
</p>

# Plane Parrallel

Plane parallel devides the full plane into multiple sub-plane, and then scatters to different devices. Thus, each device holds a part of the full plane. Apart from the plane, the remaining modules are replicated across all devices, and their parameters are shared among all devices.

During training, when a point sampled along the ray is fed into the model, it is first assigned to the sub-plane and the corresponding device based on its x-y coordinate. and then inferred on the device to get its rgbσ value. After inferring a batch of points, the rgbσ value are gathered across all devices.

After training with plane parallel, the weights of all sub-planes can be merged into a full plane. In this way, the model trained with plane parallel can be rendered just like a single model.

Alternatively, the plane weights keep seperate without merge, then the model is rendered just follows the infering way in training.

<p align="center">
<picture>
    <img src="https://github.com/FlushingCat/LandMark_Media_Content/blob/main/plane.png?raw=true" width="600">
</picture>
</p>

# Channel Parrallel

In Channel Parallel, both feature plane and feature line are sharded along the channel dimension. The features computed by grid_sample are also sharded along the channel dimension. Then we can get a full-channel feature if we do gather/all-gather on these sharded features.

<p align="center">
<picture>
    <img src="https://github.com/FlushingCat/LandMark_Media_Content/blob/main/channel.png?raw=true" width="300">
</picture>
</p>
