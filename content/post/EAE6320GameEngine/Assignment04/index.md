+++
author = "Yan Liu"
title = "Game Engine II Assignment04"
date = "2024-09-20"
description = "The details of assignment 04 for eae6320."
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

### Default

<img src="Honeycam 2024-09-20 00-28-18.gif" alt="Honeycam 2024-09-20 00-28-18" style="zoom:67%;" />



### Hide Mesh

> Press “F1”

<img src="Honeycam 2024-09-20 00-29-09.gif" alt="Honeycam 2024-09-20 00-29-09" style="zoom: 67%;" />



### Change Effect

> Press "F2"

<img src="Honeycam 2024-09-20 00-29-30.gif" alt="Honeycam 2024-09-20 00-29-30" style="zoom:67%;" />



## Background Color

Using **UpdateBackgroundColor** function and pass R,G,B,A separately

~~~c++
void eae6320::cMyGame::SubmitDataToBeRendered(const float i_elapsedSecondCount_systemTime, const float i_elapsedSecondCount_sinceLastSimulationUpdate)
{
	//......
	
	//animated background
	float simulateTime = static_cast<float>(GetElapsedSecondCount_simulation());
	float r = (std::cos(9.0f * simulateTime) * 0.1f) + 0.15f;
	float g = (std::sin(2.0f * simulateTime) * 0.1f) + 0.15f;
	float b = (-std::cos(5.0f * simulateTime) * 0.2f) + 0.25f;
	Graphics::UpdateBackgroundColor(r, g, b, backgroundColor.a);
	
	//......
}
~~~



## Mesh/Effect Pair

Initialize mesh and effect first 

~~~c++
eae6320::cResult eae6320::cMyGame::Initialize()
{
	//...
    
	Graphics::CreateMesh(vertexData01, indexData01, 7, 9, mesh01);
	Graphics::CreateMesh(vertexData02, indexData02, 4, 6, mesh02);
	Graphics::CreateEffect("data/Shaders/Vertex/standard.shader", "data/Shaders/Fragment/animatedColor.shader", effect01);
	Graphics::CreateEffect("data/Shaders/Vertex/standard.shader", "data/Shaders/Fragment/standard.shader", effect02);

    //...
}
~~~

Using **BindMeshWithEffect** function to show mesh with specific effect

~~~C++
void eae6320::cMyGame::SubmitDataToBeRendered(const float i_elapsedSecondCount_systemTime, const float i_elapsedSecondCount_sinceLastSimulationUpdate)
{
	//......
	
	Graphics::BindMeshWithEffect(mesh01, effect01);
	if (isDiffShader) 
	{
		Graphics::BindMeshWithEffect(mesh01, effect01);
	}
	else 
	{
		Graphics::BindMeshWithEffect(mesh01, effect02);
	}
	if (isShow) 
	{
		Graphics::BindMeshWithEffect(mesh02, effect02);
	}
	
    //......
}
~~~



## Mesh Size

### Data Members

Considering the variable used for reference counting, the data members of Mesh class should like this:

- x64 platform

~~~c++
// Variables
uint16_t m_referenceCount = 1;    // 2 bytes
unsigned int s_vertexCount = 0;   // 4 bytes
unsigned int s_indexCount = 0;    // 4 bytes
cVertexFormat* s_vertexFormat = nullptr;   // 8 bytes(x64)
ID3D11Buffer* s_vertexBuffer = nullptr;    // 8 bytes(x64)
ID3D11Buffer* s_indexBuffer = nullptr;     // 8 bytes(x64)
~~~

- x86 platform

~~~c++
// Variables
uint16_t m_referenceCount = 1;    // 2 bytes
unsigned int s_vertexCount = 0;   // 4 bytes
unsigned int s_indexCount = 0;    // 4 bytes
GLuint s_vertexBufferId = 0;  // 4 bytes
GLuint s_indexBufferId = 0;   // 4 bytes
GLuint s_vertexArrayId = 0;   // 4 bytes
~~~



### Memory Analysis

By debugging, it is interesting to find the size of Mesh in x64 platform is not 34, which is the sum of the size of each variables, but is 40, while it is not 22 but 24 in x86 platform. The reason for this situation is memory alignment.



**Why it couldn't be smaller?**

To draw geometry, you must let it know the data size, so you have to store the count info of vertex and index. Direct3D and OpenGL must have the data needed to draw geometry so the related data cannot be removed.



## Effect Size

### Data Members

Considering the variable used for reference counting, the data members of Effect class should like this:

- x64 platform

~~~c++
uint16_t m_referenceCount = 1;        //2 bytes
cShader* s_vertexShader = nullptr;    //8 bytes(x64)
cShader* s_fragmentShader = nullptr;  //8 bytes(x64)
cRenderState s_renderState;           //32 bytes
~~~

- x86 platform

~~~c++
uint16_t m_referenceCount = 1;        //2 bytes
cShader* s_vertexShader = nullptr;    //4 bytes(x86)
cShader* s_fragmentShader = nullptr;  //4 bytes(x86)
cRenderState s_renderState;           //1 bytes
GLuint s_programId = 0;               //4 bytes
~~~



### Memory Analysis

Like Mesh, because of memory alignment, the size of effect is 56 in x64 platform and 20 in x86 platform



**Why it couldn't be smaller?**

To bind shading data, you must know vertex shader and fragment shader, so they must be stored as variables. And OpenGL must have the data needed so the related data cannot be removed.



## sDataRequiredToRenderAFrame

### Struct

~~~c++
struct sColor
{
	float r = 0.f;
	float g = 0.f;
	float b = 0.f;
	float a = 1.f;
};

struct sDataRequiredToRenderAFrame
{
	eae6320::Graphics::ConstantBufferFormats::sFrame constantData_frame;
	sColor backgroundColor;
	std::vector<std::pair<eae6320::Graphics::Mesh*&, eae6320::Graphics::Effect*&>> meshEffectPairs;
};
~~~



> Handle Memory Constraint

I make a max size constraint(99) of vector to limit the number of mesh effect pair

~~~c++
void eae6320::Graphics::BindMeshWithEffect(Mesh*& mesh, Effect*& effect)
{
	//......
    
	if (meshEffectPairs.size() > 99) 
	{
		EAE6320_ASSERTF(false, "the mesh number over the limitation, the limitation of mesh in one frame is 99");
		Logging::OutputError("the mesh number over the limitation, the limitation of mesh in one frame is 99");
	}
    
    //......
}
~~~



### Memory Cost

By calling sizeof(), the size of sDataRequiredToRenderAFrame is 192 in x64 platform and 176 in x86 platform. Considering sizeof(vector) can only get a constant value, the real memory cost would be higher. 



The member type of vector is `std::pair<eae6320::Graphics::Mesh*&, eae6320::Graphics::Effect*&>`, so the size of each member would be 16 in x64 platform and 8 in x86 platform. As I set the max size of vector is 99, so the max memory cost of sDataRequiredToRenderAFrame would be 1776 in x64 and 968 in x86.



## Game Sample

Download and have a try: [MyGame](https://drive.google.com/uc?export=download&id=1w6lXHrSBh0SmcRRlMoJapI5dylDw5bmn)
