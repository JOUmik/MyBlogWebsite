+++
author = "Yan Liu"
title = "Game Engine II Engine System Update 01"
date = "2024-10-25"
description = "The details of Engine System Update 01 for eae6320."
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

## Collision Component

In my collision system, I made a **BaseCollisionComponent** class and two child classes **BoxColliisionComponent** and **SphereCollisionComponent**. I use an enum class to identify the shape of the collision component. Users utilising the collision component can enable/disable the hit event and overlap event based on their requirements by calling the functions **EnableHitEvent(bool)** and **EnableOverlapEvent(bool)**. 

~~~c++
enum class CollisionType
{
    None,
    Box,
    Sphere
};
~~~



## Delegate System

Similar to unreal, I made a delegate system for collision system to broadcast the events(**OnComponentHit, OnComponentBeginOverlap, OnComponentEndOverlap**), this design make the whole system more flexible and do not need to care what users want to do.

~~~c++
template<typename... Args>
class Delegate 
{
public:
    using CallbackType = std::function<void(Args...)>;

    void Add(CallbackType callback) 
    {
        callbacks.push_back(callback);
    }

    void RemoveAll() 
    {
        callbacks.clear();
    }

    void Broadcast(Args... args) 
    {
        for (auto& callback : callbacks) 
        {
            callback(args...);
        }
    }

private:
    std::vector<CallbackType> callbacks;
};

class BaseCollisionComponent
{
public:
    Delegate<const BaseCollisionComponent&> OnComponentHit;
    Delegate<const BaseCollisionComponent&> OnComponentBeginOverlap;
    Delegate<const BaseCollisionComponent&> OnComponentEndOverlap;
    
    //others
}
~~~



## Collision Manager

I made a CollisionManager class, that uses the Singleton pattern. It is responsible for updating the positions of all components and comparing the detected overlap state in each frame with the state of the previous frame to ensure the correct triggering of OverlapRegister and OverlapEnd events, as well as the Hit event. To achieve this, I used two sets to cache the overlap pairs of the current and previous frames (current Overlaps and previous Overlaps).



In order to improve efficiency and avoid duplicate detection of the same collision pairs in the same frame, I made a collisionCache to cache the collision pairs that have already been detected in the current frame. This way, even if the component enters the Update process multiple times in the same frame, it will not repeatedly detect collisions that have already been processed.

> **The Way I Do**
>
> **Cache mechanism:** Add a collisionCache in CollisionManager to store collision pairs that have been detected in each frame.
>
> **Query cache before detection:** Before each collision detection, check if collisionCache already contains the current component pair.
> If it already exists, skip detection. If it does not exist, perform collision detection and add the current component pair to collisionCache.
>
> **Clear cache per frame:** Clear collisionCache at the end of each frame for re detection in the next frame.



## Location Update

In order to achieve smooth movement and position update of components, I use linear interpolation (Lerp) to calculate the smooth transition from the current position to the target position, ensuring that collision detection can be accurately performed at each interpolation step. By calculating the interpolation points on the original and target position paths, it prevents being missed when there are other components on the path



## What's next

I would make the BVH structure to speed up the collision detection efficiency. After I finish it, I will make a demo to test whether everything works great.
