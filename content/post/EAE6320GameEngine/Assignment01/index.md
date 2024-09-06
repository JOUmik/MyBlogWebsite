+++
author = "Yan Liu"
title = "Game Engine II Assignment01"
date = "2024-08-26"
description = "The details of assignment 01 for eae6320."
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

<!--more-->

## Gif of My Game

<img src=".\animatedcolor.gif" alt="animatedcolor" style="zoom:67%;" />



## Log of My Game

<img src=".\log.png" />



## Add Reference

The project needs to add reference to Graphics is **Application**.

I found **ShaderBuilder** has mentioned graphics namespace but don't need to add reference to Graphics. The reason is that ShaderBuilder only uses Graphics::ShaderType and it is an enum, which means it doesn't require any function from Graphics. 



## Optional Challenges

### Slow time

Add responses to keyboard events in cMyGame.cpp, I use **space** to slow the time

~~~c++
void eae6320::cMyGame::UpdateBasedOnInput()
{
	// Is the user pressing the ESC key?
	if ( UserInput::IsKeyPressed( UserInput::KeyCodes::Escape ) )
	{
		// Exit the application
		const auto result = Exit( EXIT_SUCCESS );
		EAE6320_ASSERT( result );
	}

	// If the user press space, slow the time
	if (UserInput::IsKeyPressed(UserInput::KeyCodes::Space))
	{
		//slow the time
		SetTimeRate(0.2f);
	}

	// If the user release space, restore the time
	if (!UserInput::IsKeyPressed(UserInput::KeyCodes::Space))
	{
		//restore the time
		SetTimeRate(1.0f);
	}
}
~~~

Add function to iApplication files to set time rate

~~~c++
/*
* iApplication.h
*/
public:
	void SetTimeRate(const float rate);

/*
*iApplication.cpp
*/
//Set the time rate
void eae6320::Application::iApplication::SetTimeRate(const float rate)
{
	m_simulationRate = rate;
}
~~~





## Thoughts

### organization

The organization of this project is very in line with my previous coding habits, distinguishing different functional modules, which not only makes it easy to find and modify, but also makes the logic clearer.

### code style

I think it's a great thing to name every function and variable based on its actual meaning, which not only facilitates future modifications but also makes it easier for readers to understand. Meanwhile, there are numerous detailed comments in the code that make understanding relatively easy.

I have also found that unnamed namespace is also used in the code, and I think this is a very good thing to prevent functions with naming conflicts.

### confusion

The project configuration has really troubled me for a long time. I haven't had a similar experience before, but fortunately, after communicating with my classmates and step-by-step troubleshooting the configuration, it was resolved. I would like to thank **Lehan Li** for her help, and I am really grateful for taking the time to help me troubleshoot the configuration problem. 

My previous operation was to build ExempleGame first and then build BuildExempleGameAssets. This resulted in some lib files used by BuildExempleGameAssets not being generated, as ExempleGame did not reference these libs. Just build the solution directly and there would be no problem. Now it seems a tiny problem, but really stuck me for a long time.



## Expectation

I am really excited about how to build game engines. When using the UE engine, I often marvel at the powerful features and wonder how they are implemented. However, due to my own ability issues, reading the UE source code has always been a confusing thing for me. I hope to learn a series of graphics knowledge related to game engines, underlying design frameworks, performance optimization algorithms used, and more through this semester's courses



## Time Cost

This assignment took me over 10 hours, with the majority of time spent on resolving configuration issues



## Game Sample

Download and have a try: [MyGame](https://drive.google.com/uc?export=download&id=1Jp8nIzYdjSdQOfs63wyZ2yikuC6ybzke)
