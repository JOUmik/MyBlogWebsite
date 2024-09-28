+++
author = "Yan Liu"
title = "Game Engine II Assignment05"
date = "2024-09-27"
description = "The details of assignment 05 for eae6320."
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

> Press “WASD” to move object

> Press "F3" to change mesh

> Press "$\uparrow$, $\rightarrow$, $\downarrow$, $\leftarrow$" to move camera

<img src="Honeycam 2024-09-27 20-16-34.gif" alt="Honeycam 2024-09-27 20-16-34" style="zoom:67%;" />



## Actor

### Introduction

Like Unreal, I use **AActor** to represent the objects that can be put in the scene.

~~~c++
class AActor : public UObject, public eae6320::Graphics::cRenderableObject
{
public:
	AActor(eae6320::Graphics::Mesh* i_mesh, eae6320::Graphics::Effect* i_effect);
	~AActor();

	virtual void CleanUp();
};
~~~

If the user wants to create an object in the scene, he needs to create an actor instance first, then uses the functions provided by Actor and its parent classes.

~~~c++
//.h
//Actors
eae6320::GameFramework::AActor* house = nullptr;
eae6320::GameFramework::AActor* chimney = nullptr;

//.cpp
//Actor
house = new eae6320::GameFramework::AActor(mesh01, effect01);
chimney = new eae6320::GameFramework::AActor(mesh02, effect02);
~~~

Now the functions the user needs mainly in its parent class **cRenerabaleObject**, a class in Graphics namespace that provides the functions to change mesh or effect and submit necessary data to draw.

~~~c++
class cRenderableObject 
{
public:
	eae6320::Physics::sRigidBodyState* rigidBodyState = nullptr;

	void CleanUp();

	void ChangeMesh(Mesh* new_mesh);
	void ChangeEffect(Effect* new_effect);

	void SubmitMeshWithEffectToDraw(const float i_elapsedSecondCount_sinceLastSimulationUpdate);

protected:
	inline void SetMesh(Mesh* i_mesh) 
	{
		mesh = i_mesh; 
		mesh->IncrementReferenceCount();
	}
	inline void SetEffect(Effect* i_effect) 
	{ 
		effect = i_effect; 
		effect->IncrementReferenceCount();
	}

private:
	Mesh* mesh = nullptr;
	Effect* effect = nullptr;
};
~~~

And as shown in the code, it has the variable of **sRigidBodyState** structure so it can send data to change object's position.



### Change Mesh and Effect

As shown in the code of 2.1, if the user needs to change the mesh or effect of one actor, just call **SetMesh** or **SetEffect**

~~~c++
//cMyGame.cpp
void eae6320::cMyGame::SwitchShader()
{
	if (!isDiffShader) 
	{
		house->ChangeEffect(effect02);
	}
	else 
	{
		house->ChangeEffect(effect01);
	}
}

void eae6320::cMyGame::SwitchMesh()
{
	if (isCubeMesh) 
	{
		house->ChangeMesh(mesh03);
	}
	else 
	{
		house->ChangeMesh(mesh01);
	}
}
~~~



### Submit Data to Render

As shown in the code of 2.1, if the user needs to render an actor, all he needs to do is call **SubmitMeshWithEffectToDraw**, which is a function of **cRenderableObject** class and be inherited by **AActor**.

~~~c++
void eae6320::cMyGame::SubmitDataToBeRendered(const float i_elapsedSecondCount_systemTime, const float i_elapsedSecondCount_sinceLastSimulationUpdate)
{
	//......
    
    //Draw Actors
    {
        house->SubmitMeshWithEffectToDraw(i_elapsedSecondCount_sinceLastSimulationUpdate);

        if (isShow)
        {
            chimney->SubmitMeshWithEffectToDraw(i_elapsedSecondCount_sinceLastSimulationUpdate);
        }
    }
        
    //......
}
~~~



## Camera Actor

I make a class called **ACameraActor** to represent camera, this class store all the necessary data for camera.

~~~c++
class ACameraActor
{
public:
	ACameraActor();
	~ACameraActor() = default;
	void SubmitCameraData(const float i_secondCountToExtrapolate);
	void Update(const float i_elapsedSecondCount_sinceLastUpdate);
	void CleanUp();

	inline void SetVelocity(Math::sVector velocity) { rigidBodyState->velocity = velocity; }
	inline Math::sVector GetVelocity() { return rigidBodyState->velocity; }

	inline void SetPosition(Math::sVector position) { rigidBodyState->position = position; }

	float verticalFieldOfView_inRadians;
	float aspectRatio;
	float z_nearPlane;
	float z_farPlane;

private:
	eae6320::Physics::sRigidBodyState* rigidBodyState = nullptr;
};
~~~

Like Actor, I also use **sRigidBodyState** to control camera's movement.



## Player Controller

### Introduction

Like Unreal, I put the control of the camera into the player controller, and this is all the player controller does for now.

~~~c++
class APlayerController 
{
public:
	void Update(const float i_elapsedSecondCount_sinceLastUpdate);
	void SetCurrentCamera(ACameraActor* newCamera);
	void SubmitDataToGraphics(const float i_secondCountToExtrapolate);

private:
	ACameraActor* currentCamera = nullptr;
};
~~~



### Change Camera

If the user wants to change the camera, just call **SetCurrentCamera**



### Submit Camera Data

To submit camera data, call **SubmitDataToGraphics**

~~~c++
void eae6320::cMyGame::SubmitDataToBeRendered(const float i_elapsedSecondCount_systemTime, const float i_elapsedSecondCount_sinceLastSimulationUpdate)
{
    // Player Controller send the binded camera date to graphics
	{
		playerController->SubmitDataToGraphics(i_elapsedSecondCount_sinceLastSimulationUpdate);
	}
}
~~~



## sDataRequiredToRenderAFrame

### Structure

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
    std::vector<eae6320::Graphics::ConstantBufferFormats::sDrawCall> constantData_drawCalls;
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

By calling sizeof(), the size of sDataRequiredToRenderAFrame is 224 in x64 platform and 192 in x86 platform. Considering sizeof(vector) can only get a constant value, the real memory cost would be higher. 



The member type of  meshEffectPairs vector is `std::pair<eae6320::Graphics::Mesh*&, eae6320::Graphics::Effect*&>`, so the size of each member would be 16 in x64 platform and 8 in x86 platform. The member type of constantData_drawCalls is `eae6320::Graphics::ConstantBufferFormats::sDrawCall`, so the size of each member would be 64 in both platform. 



As I set the max size of vector is 99, so the max memory cost of sDataRequiredToRenderAFrame would be 8112 in x64 and 7304 in x86.



## Extrapolation/Prediction

By extrapolating the position actor would, you can get a smooth movement. The way to make it is using elapsedSecondCount_sinceLastSimulationUpdate. It can be used to smoothly interpolate the position changes of objects based on the time difference between the previous physical simulation and the current rendering. This interpolation compensates for the jitter caused by the different update frequencies of rendering and simulation, ensuring smoother motion of objects between rendered frames.



## Game Sample

Download and have a try: [MyGame](https://drive.google.com/uc?export=download&id=19zu03WJIiVsU6a5MfG1N0DvXQG1cQZfM)
