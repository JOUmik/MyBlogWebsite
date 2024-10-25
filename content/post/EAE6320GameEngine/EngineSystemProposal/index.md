+++
author = "Yan Liu"
title = "Game Engine II Engine System Proposal"
date = "2024-10-25"
description = "The details of Engine System Proposal for eae6320."
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

## Introduction

The Engine System I want to make is a **Collision System** which handles interactions between actors in the game. The current implementation is designed to be simple and efficient, supporting **Box** **and Sphere Components** for collision detection. It uses a **Bounding Volume Hierarchy (BVH)** with **Axis-Aligned Bounding Boxes (AABB)** to optimize collision checks, making it suitable for basic physics and actor interaction in a 3D environment.

 

## Key Features

- **Box and Sphere Collision**: The system supports collision detection for actors defined as boxes or spheres. These are the simplest shapes to work with and are commonly used for fast collision detection.
- **BVH**: The system uses BVH with AABB to speed up collision queries. This hierarchical structure helps in minimizing the number of collision checks by narrowing down potential collisions between actors.
- **Potential Future Support**: If time allows, support for **Oriented Bounding Boxes (OBB)** and **Capsule Components** may be added, which would provide more efficient and accurate collision for actors.

 

## How to Use the Features

- **Classes and Interfaces**: 

​	BoxComponent and SphereComponent: These define the shape and size of the colliders.

​	BVHNode: Represents nodes in the bounding volume hierarchy, helping to group and organize collision actors for efficient processing.

​	CollisionManager: Manages the overall collision system, running checks and updating collision status between actors.

 

- **Functions**:

​	UpdatePosition(): Users can update collision components manually to make them follow actors.

​	TryMove(): Returns whether an actor can move to one specific position. If it is collided with another actor, return false.

​	CheckCollision(): Returns whether two actors are colliding based on their components.

​	UpdateBVH(): Reorganizes the BVH tree to reflect changes in actor positions or sizes.

 

## What Will Be Implemented

- **Box and Sphere Components** for collision detection.

- A **BVH (AABB)** system to optimize collision queries.
- Simple API functions to add components and check collisions.

 

## Challenges and Stretch Goals

- Challenges: Managing the performance of the BVH system, especially in complex scenes with many actors, could be tricky. Ensuring that the hierarchy updates efficiently when actors move or change.
- Stretch Goals: If time permits, adding support for more complex shapes like Capsule would improve the accuracy and versatility of the system. And adding OBB as an option for BVH would improve its efficiency.
