+++
author = "Yan Liu"
title = "Game Engine II Assignment03"
date = "2024-09-13"
description = "The details of assignment 03 for eae6320."
tags = [
    "EAE6320"
]
categories = [
    "game engine"
]
series = ["Themes Guide"]
aliases = ["migrate-from-jekyl"]
image = "engine.jpg"

+++

## Gif of My Game

<img src="AnimatedHouse.gif" alt="animatedcolor" style="zoom:67%;" />

## Graphics.cpp platform-independent

My method of making Graphics platform independent is to derive a View class from Graphics. This class would handle clearing previous images, initializing view, swapping images to the front buffer and cleaning up relevant data.



## Clear the back buffer color

As mentioned above, Graphics file cleans the back buffer color by using the function provided by view class.

~~~c++
//Declare variable
eae6320::Graphics::View view;

//In RenderFrame function
view.ClearPreviousImage(constantData_frame.g_elapsedSecondCount_simulationTime);
~~~



By passing simulation time to ClearPreviousImage function and doing the similar work as the fragment shader to make the background color animate.

~~~c++
//Take Direct3D as an example
float r = (std::cos(9.0f * g_elapsedSecondCount_simulationTime) * 0.1f) + 0.15f;
float g = (std::sin(2.0f * g_elapsedSecondCount_simulationTime) * 0.1f) + 0.15f;
float b = (-std::cos(5.0f * g_elapsedSecondCount_simulationTime) * 0.1f) + 0.15f;
const float clearColor[4] = { r, g, b, 1.0f };
direct3dImmediateContext->ClearRenderTargetView(s_renderTargetView, clearColor);
~~~



And the result has been shown on "Gif of My Game".



## Effect

### Initialize

By passing vertex shader path and fragment shader path to initialize an effect

~~~c++
if (!(result = effect01.InitializeShadingData("data/Shaders/Vertex/standard.shader", "data/Shaders/Fragment/animatedColor.shader")))
{
	EAE6320_ASSERTF(false, "Can't initialize Graphics without the shading data");
	return result;
}
~~~



### Memory Analysis

~~~c++
cResult InitializeShadingData(const std::string& vertexShaderPath, const std::string& fragmentShaderPath);
~~~

By debugging, it is interesting to find the size of string type is different between x86 and x64. The reason for it is that the size of pointer is different in two platforms. Normally, the size of pointer in x86 platform is 4 bytes while it is 8 bytes in x64 platform. And for that, the memory alignment is also different. As a result, the size of string in x86 platform is 28 bytes and in x64 platform is 40 bytes. 



Considering two string variables are needed, a single effect would take up 56 bytes in x86 platform and 80 bytes in x64 platform.



**Why it couldn't be smaller?**

Because two strings pointing to vertex shader path and fragment shader path respectively are necessary.



## Mesh

### Initialize

By passing vertex data, index data and the size of them to initialize a mesh

~~~c++
if (!(result = mesh01.InitializeGeometry(vertexData01, indexData01, 7, 9)))
{
	EAE6320_ASSERTF(false, "Can't initialize Graphics without the geometry data");
	return result;
}
~~~



### Memory Analysis

~~~c++
cResult InitializeGeometry(VertexFormats::sVertex_mesh* vertexData, uint16_t* indexData, unsigned int vertexCount, unsigned int indexCount);
~~~

Because the vertex data and index data are passed as pointers, and as mentioned in effect memory analysis, the size of pointer is different in the two platforms. So two pointers and two unsigned int type variables would take up 16 bytes in x86 platform and 24 bytes in x64 platform.



**Why it couldn't be smaller?**

To initialize geometry, you must let it know the size of data, so you have to pass the count info. And considering the performance, using pointers point the address of vertex data and index data.



## Game Sample

Download and have a try: [MyGame](https://drive.google.com/uc?export=download&id=1ccMnAYkwAvyNfuQxdfGzzzyN2wxcBIHG)
