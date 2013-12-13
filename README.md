-------------------------------------------------------------------------------
Deferred Shader
-------------------------------------------------------------------------------
Fall 2013
-------------------------------------------------------------------------------
![fullscene](/renders/FullHallAllTextures.PNG "Finished Rendering")

Youtube Video of Rendering:
<dl>
<a href="http://www.youtube.com/watch?feature=player_embedded&v=kvQ3dNG4Mdg
" target="_blank"><img src="http://img.youtube.com/vi/kvQ3dNG4Mdg/0.jpg" 
alt="Youtube Video of Rendering Process" width="480" height="360" border="10" /></a>
</dl>

NOTE:
-------------------------------------------------------------------------------
This project requires a CUDA-capable graphics card, as well as OpenNI and an
Xbox Kinect to run a live demonstration.

-------------------------------------------------------------------------------
BACKGROUND:
-------------------------------------------------------------------------------
Previous work has demonstrated the diverse capabilities of RGBD cameras, from
generating highly accurate 3D surface models to reliable 3D pose estimation.
However, many algorithms attempt to store the generated environment as a RGB 3D
point cloud, which is not easily adaptable to dynamic environments, requires
very large quantities of memory to store large environments, and provides no
intuition to higher perception processes about distinct objects beyond a
volumetric approximation. Other approaches have been able to store and merge
the surface data more efficiently, but still regard the environement as a
unified whole rather than discrete objects. By extracting meaningful geometry
from the RGB-D in the form of triangle meshes instead, a large number of
advantages can be realized.

* High storage efficiency
* Natural low level object segmentation
* Easy to manipulate, modify, and render in real time
* Efficient and easy to process intuition of geometry that higher cognitive functions can use for object recognition and manipulation tasks.
* Straightforward tradeoff between simplicity and accuracy with mesh resolution

-------------------------------------------------------------------------------
CONTENTS:
-------------------------------------------------------------------------------
The Project6 root directory contains the following subdirectories:
	
* CUDA-Mesh/ is ...
* Recorder-Viewer/ is ...
* algorithm-python/ is ...
* shared/ contains glew

-------------------------------------------------------------------------------
CODE TOUR
-------------------------------------------------------------------------------

The overall image processing line is shown below. First, an RGB frame and a depth frame are pulled from the Kinect and shipped to the GPU for processing. A world-space point cloud is then generated from the RGBD data, and a neighborhood-based estimate of the point normals is then extracted for later processing. Finally, the point cloud is triangulated and the generated mesh is passed to OpenGL where a variety of rendering options are implemented.

![Image Processing Pipeline](/docs/diagrams/ImageProcessingPipeline.png "Image Processing Pipeline")
![Framework Layout](/docs/diagrams/FrameworkLayout.png "Framework Layout")
![Program Flow](/docs/diagrams/ProgramFlow.png "Program Flow")
![OpenGL Pipeline](/docs/diagrams/OpenGLPipeline.png "OpenGL Pipeline")

-------------------------------------------------------------------------------
CONTROLS
-------------------------------------------------------------------------------

Stage 1 samples model textures renders the scene geometry to the G-Buffer
* pass.vert
* pass.frag

Stage 2 renders the lighting passes and accumulates to the P-Buffer
* shade.vert
* ambient.frag
* point.frag
* diagnostic.frag

Stage 3 renders the post processing
* post.vert
* post.frag

Keyboard controls
[keyboard](https://github.com/cboots/Deferred-Shading/blob/master/base/src/main.cpp#L1178):
This is a good reference for the key mappings in the program. 
WASDQZ - Movement
X - Toggle scissor test
R - Reload shaders
1 - View depth
2 - View eye space normals
3 - View Diffuse color
4 - View eye space positions
5 - View lighting debug mode
6 - View Specular Mapping
7 - View Only Bloomed Geometry
0 - Standard view

x - Toggle Scissor Test
r - Reload Shaders
p - Print camera position to console
j - Toggle timing measurements to console (averaged since last reset)
SPACE - Reset timing averages

Shift-L - Toggle Bloom
Shift-T - Toggle Toon Shading
Shift-D - Toggle Diffuse Mapping
Shift-S - Toggle Specular Mapping
Shift-B - Toggle Bump Mapping
Shift-M - Toggle Transparency Masking

c - Reset  diffuse color to default
t - Change diffuse color to texture coordinate visualization
h - Overlay diffuse color with visualization of available textures
b - Change diffuse color to bump map visualization if available
m - Change diffuse color to white if mask texture is available

-------------------------------------------------------------------------------
Features:
-------------------------------------------------------------------------------

* Renders .obj files with support for .mtl files with
  * Diffuse Textures
  * Specular Textures
  * Height Maps (Bump Mapping)
  * Texture Masking
 
* Bloom (Not seperable, very inefficient)
![NoBloom](/renders/LampNoBloom.PNG "Without Bloom")
![Bloom](/renders/LampWithBloom.PNG "With Bloom")

* "Toon" Shading (with basic silhouetting and color quantization)
![Toon](/renders/ToonShadingNoColor.PNG "Toon Shading B/W")
![Toon](/renders/FullHallToon.PNG "Toon Shading")

* Point light sources with specular

![Point Specular](/renders/PointLightSpeculars.PNG "Point Light Speculars")


-------------------------------------------------------------------------------
SCREENSHOTS
-------------------------------------------------------------------------------

3D Point Cloud Normals Above.PNG:
![3DPointCloudNormalsAbove.PNG](/docs/screenshots/3DPointCloudNormalsAbove.PNG "3DPointCloudNormalsAbove.PNG")
ColorfulOverlay.PNG:
![ColorfulOverlay.PNG](/docs/screenshots/ColorfulOverlay.PNG "ColorfulOverlay.PNG")
ImprovedNormals.PNG:
![ImprovedNormals.PNG](/docs/screenshots/ImprovedNormals.PNG "ImprovedNormals.PNG")
MeshNormals.PNG:
![MeshNormals.PNG](/docs/screenshots/MeshNormals.PNG "MeshNormals.PNG")
3D Point Cloud Normals.PNG:
![3DPointCloudNormals.PNG](/docs/screenshots/3DPointCloudNormals.PNG "3DPointCloudNormals.PNG")
DepthDataGUI.PNG:
![DepthDataGUI.PNG](/docs/screenshots/DepthDataGUI.PNG "DepthDataGUI.PNG")
Mesh.PNG:
![Mesh.PNG](/docs/screenshots/Mesh.PNG "Mesh.PNG")
Window.PNG:
![Window.PNG](/docs/screenshots/Window.PNG "Window.PNG")
ChairPointCloudNormals.PNG:
![ChairPointCloudNormals.PNG](/docs/screenshots/ChairPointCloudNormals.PNG "ChairPointCloudNormals.PNG")
FaceFilled.PNG:
![FaceFilled.PNG](/docs/screenshots/FaceFilled.PNG "FaceFilled.PNG")
MeshFile.PNG:
![MeshFile.PNG](/docs/screenshots/MeshFile.PNG "MeshFile.PNG")
python-normals.png:
![python-normals.png](/docs/screenshots/python-normals.png "python-normals.png")
ColorHairs.PNG:
![ColorHairs.PNG](/docs/screenshots/ColorHairs.PNG "ColorHairs.PNG")
Hairs.PNG:
![Hairs.PNG](/docs/screenshots/Hairs.PNG "Hairs.PNG")
MeshHead.PNG:
![MeshHead.PNG](/docs/screenshots/MeshHead.PNG "MeshHead.PNG")
AllStages.PNG:
![AllStages.PNG](/docs/screenshots/AllStages.PNG "AllStages.PNG")
MeshFace.PNG:
![MeshFace.PNG](/docs/screenshots/MeshFace.PNG "MeshFace.PNG")
Points.PNG:
![Points.PNG](/docs/screenshots/Points.PNG "Points.PNG")

-------------------------------------------------------------------------------
PERFORMANCE EVALUATION
-------------------------------------------------------------------------------

Our point normals kernel was implemented as follows. A window radius is first
specified as an algorithm parameter. For each point, we loop through its
neighboring points in screen space in the square window specified by the
radius, and pair it with a screen-space orthogonal point at the same radius. If
both points are within a specified radius from the center point in world space,
we take the cross product to compute the normal, which is then flipped if
pointing away from the camera. If sufficiently many valid normals are found, we
average them to produce the final normal estimate, otherwise we discard the
point.

To improve the runtime of the point normals kernel, we reimplemented the
algorithm using shared memory. In the shared memory implementation, all points
in given thread block are first loaded into shared memory, along with the
points lying within the specified neighborhood radius of the edges of the
thread block, and the distance and cross product calculations are then
performed using shared memory access. The results of the shared memory
optimization on kernel runtime are shown for a range of window radii using a
thread block size of 8x8.

![KernelRuntime](/docs/performance/SharedVsGlobalRuntime.png "Kernel Runtime")
![FPS](/docs/performance/SharedVsGlobalFPS.png "FPS")

As demonstrated, the shared memory optimization reduced the kernel runtime by
approximately a factor of 2. The impact on the overall FPS was less dramatic,
though still pronounced, due to the time spent in the rendering pipeline.

All testing for this project was conducted on the following hardware:
* CPU: Intel Core i5-2450M, 2.5GHz 6GB (Windows 8, 64-bit OS)
* GPU: NVIDIA GeForce GT 525M with 2 SM's:

-------------------------------------------------------------------------------
ACKNOWLEDGEMENTS
-------------------------------------------------------------------------------

REMEMBER TO ACKNOWLEDGE LIBRARIES

