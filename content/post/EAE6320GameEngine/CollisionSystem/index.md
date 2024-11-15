+++
author = "Yan Liu"
title = "Game Engine II Collision System"
date = "2024-11-14"
description = "The details of Collision System."
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

## What the system does

This system is a collision system, it provides two different collision components, **SphereCollisionComponent** and **BoxCollisionComponent**. You can use this system to generate **OnComponentHit** event, **OnComponentBeginOverlap** event and **OnComponentEndOverlap** event.

<img src="Honeycam 2024-11-14 10-35-31.gif" alt="Honeycam 2024-11-14 10-35-31" style="zoom:67%;" />

This system uses the **BVH** structure and some other methods to reduce the performance cost. It also uses linear interpolation and binary search to avoid tunneling happen when the actor moves too fast.



## How to use the system

### Set Up

The set up is very easy:

1. Download the Zip file I provided and unzip
2. Put it to your Engine folder
3. Set the correct props for Collision system just like what you did for Graphics system
4. Add Math system as the reference of **Collision system**
5. Add Collision system as the reference of **MyGame**



### Collision Manager

Collision manager handles collision detection from collision component set and broadcast certain events when those events happened, it also builds BVH for those components. The way it works is pretty complicated but users only need to know how to use the interfaces provided.

Collision manager uses the **singleton** pattern, if you want to use collision manager, Call the function **CollisionManager::GetCollisionManager()** first to get the variable.

~~~c++
void AddCollisionComponent(BaseCollisionComponent& comp);
void RemoveCollisionComponent(BaseCollisionComponent& comp);
static CollisionManager* GetCollisionManager();
void Begin();
void Update();
void Destroy();
~~~

- You mush call **AddCollisionComponent()** to add the component to the component set, if it is not set to the collision set, it would not do the collision detection;

- You mush call **RemoveCollisionComponent()** before you destroy the component;

- You should call **Begin()** in the beginning of the game after you add all the components to the component set;

- You should call **Update()** in each update and call **Destroy()** when you finish the game;



### Collision Component

**SphereCollisionComponent** and **BoxCollisionComponent** are the child classes of **BaseCollisionComponent**



#### Set Size

SphereCollisionComponent:

~~~c++
inline void SetRadius(float i_radius) { radius = i_radius; }
~~~

BoxCollisionComponent:

~~~c++
inline void SetExtend(Math::sVector i_extend) { extend = i_extend; }
~~~



#### Set Initial Position

~~~c++
inline void SetPosition(Math::sVector i_position) { position = i_position; }
~~~



**Notice:** This function is only used to set the initial position. If you want to change the component's position when it moves, please see **2.3.5 Movement**



#### Collision Event

You can set the collision event of collision component to **NoCollision**, **Overlap**, or **Hit**.

~~~c++
enum class CollisionEvent 
{
    NoCollision,
    Overlap,
    Hit
};
~~~

The function you need to set the collision event is **SetCollisionEvent()**. By default, it would be set to **Hit**.

~~~c++
inline void SetCollisionEvent(CollisionEvent i_collisionEvent = CollisionEvent::Hit) { collisionEvent = i_collisionEvent; }
~~~



> ***Rules:***

| Actor A     | Actor B     | Collision Event Generated |
| ----------- | ----------- | ------------------------- |
| Hit         | Hit         | Hit Event                 |
| Hit         | Overlap     | Overlap Event             |
| Overlap     | Hit         | Overlap Event             |
| Overlap     | Overlap     | Overlap Event             |
| NoCollision | Any         | None                      |
| Any         | NoCollision | None                      |



#### Collision Component Type

You can set the component type to **Static** or **Dynamic**.

~~~c++
enum class CollisionComponentType
{
    Static,
    Dynamic
};
~~~

The function you need to set the component type is **SetCollisionComponentType()**. By default, it would be set to **Dynamic**.

~~~c++
inline void SetCollisionComponentType(CollisionComponentType i_collisionComponentType = CollisionComponentType::Dynamic) { collisionComponentType = i_collisionComponentType; }
~~~



Because the **StaticBVH** would be only built once when **CollisionManager::GetCollisionManager()->Begin()** is called, while the **DynamicBVH** would be built every time **CollisionManager::GetCollisionManager()->Update()** is called. So it is good for performance if the component type is set to **static**.



<font color = "Red">"You must set the component type to **Dynamic** if you want the component to move."</font>



#### Movement

If you want to move the component to a new position, there are some steps you need to follow.

0. Again, <font color = "Red">please make sure the component type has set to **Dynamic**</font>

1. Must call **TryMoveTo()** first.

~~~c++
void eae6320::AControlledActor::Update(const float i_elapsedSecondCount_sinceLastSimulationUpdate)
{
    if (rigidBodyState->velocity.GetLength() != 0)
    {
        SphereComp->TryMoveTo(rigidBodyState->PredictFuturePosition(i_elapsedSecondCount_sinceLastSimulationUpdate));
    }
}
~~~

2. Notify **Collision manager** to do the collision detection by calling Update(). Collision manager would decide whether components can move to the new position and set them to the correct position.

~~~c++
//Update Collision
Collision::CollisionManager::GetCollisionManager()->Update();
~~~



#### Bind Event

There are four events provided by collision component

~~~c++
Delegate<const BaseCollisionComponent&> OnComponentHit;
Delegate<const BaseCollisionComponent&> OnComponentBeginOverlap;
Delegate<const BaseCollisionComponent&> OnComponentEndOverlap;
Delegate<const Math::sVector&> UpdatePositionAfterCollision;
~~~

- **OnComponentHit** event would broadcast to the functions binded to it the info of the component it hits.
- **OnComponentBeginOverlap** event would broadcast to the functions binded to it the info of the component it just overlaps.
- **OnComponentEndOverlap** event would broadcast to the functions binded to it the info of the component it just finishes overlapping with.
- **UpdatePositionAfterCollision** event would broadcast to the functions binded to it the position of itself after collision detection.



> *Example of binding event*

~~~c++
eae6320::AControlledActor::AControlledActor(eae6320::Graphics::Mesh* i_mesh, eae6320::Graphics::Effect* i_effect)
{
    //Bind Event
    SphereComp->UpdatePositionAfterCollision.Add(this, &AControlledActor::UpdatePosition);
}

void eae6320::AControlledActor::UpdatePosition(const Math::sVector& safePosition)
{
    SetPosition(safePosition);
}
~~~

~~~c++
eae6320::AOverlapBeginTestActor::AOverlapBeginTestActor(eae6320::Graphics::Mesh* i_mesh, eae6320::Graphics::Effect* i_effect)
{
    //Bind Event
    boxComp->OnComponentBeginOverlap.Add(this, &AOverlapBeginTestActor::OnComponentBeginOverlap);
}

void eae6320::AOverlapBeginTestActor::OnComponentBeginOverlap(const Collision::BaseCollisionComponent&)
{
    //Your code here
}
~~~



## How I used what I have learned this semester

- The way to set up an engine system and make it work with other system;
- The way to make graphics debugging, thanks for that I did not stuck on any issue for a long time;
- Taking the advantage of logging system and use it to test whether the demo works as I expected;





## Anything learned or struggled

- How to build BVH;

~~~c++
struct BVHNode 
{
    BoundingBox bounds;
    std::unique_ptr<BVHNode> left;
    std::unique_ptr<BVHNode> right;
    std::vector<BaseCollisionComponent*> components;

    bool IsLeaf() const 
    {
        return left == nullptr && right == nullptr;
    }
};
~~~

- The way to use custom hash rule to store pairs in hash set;

~~~c++
// custom hash function
struct CollisionPairHash {
    std::size_t operator()(const CollisionPair& pair) const {
        return std::hash<void*>()(pair.componentA) ^ std::hash<void*>()(pair.componentB);
    }
};

std::unordered_set<CollisionPair, CollisionPairHash> currentOverlaps;
~~~

- How the delegate system works;

- The optimization methods for collision system;

- The way to handle tunneling problem;

~~~c++
Math::sVector Lerp(const Math::sVector& start, const Math::sVector& end, float t);
Math::sVector PerformBinarySearch(const Math::sVector& start, const Math::sVector& end, float low, float high, BaseCollisionComponent& comp);
~~~



## Collision System Download

Download and have a try:  [CollisionSystem](https://drive.google.com/uc?export=download&id=1ZXA0feBHdrtf_iwKjp7puEI2Nxkq9NIz)
