+++
author = "Yan Liu"
title = "Game Engine II Engine System Update 02"
date = "2024-11-07"
description = "The details of Engine System Update 02 for eae6320."
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

## Gif of My Collision System Demo

<img src="Honeycam 2024-11-05 12-28-08.gif" alt="Honeycam 2024-11-05 12-28-08" style="zoom:67%;" />

This demo shows how the collision system works, the <font color="#38A84E">**green**</font> one enabled **hit event**, <font color=red>**red**</font> one enabled **overlap begin event**,  <font color="#F7DB17">**yellow**</font> one enabled **overlap end event**



## Static/Dynamic

For better performance, I made a **CollisionComponentType** variable for each collision component:

~~~c++
enum class CollisionComponentType
{
    Static,
    Dynamic
};
~~~

If the component is set to static, then it would never move so the collision between it and other components would not be detected actively. If the component is set to dynamic and it moved in current update, it would actively detect whether there are any component collide with it and broadcast certain event.

~~~c++
// only update the dynamic components that are moving in this update
if (comp->GetCollisionComponentType() != CollisionComponentType::Dynamic || !comp->bIsMoving) continue;
~~~



## BVH

For better performance further, I implemented a BVH structure for the collision system. The structure of each BVH node is like:

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

Considering the difference of static and dynamic components, I made two variables in CollisionManager to store the BVH info:

~~~c++
std::unique_ptr<BVHNode> staticBVH;
std::unique_ptr<BVHNode> dynamicBVH;
~~~

The staticBVH would be only built once in the very beginning of the game because static components never move. The dynamicBVH would be built in each update to make sure the info is correct.



## Demo Implement

To test whether the collision system works as I expected, I made a demo as shown in **1. Gif of My Collision System Demo**. In this demo, I made three different actor classes, they are used to test different collision event(**hit, overlap begin, overlap end**). In each class, I used delegate system to bind the function with certain event, so when certain event happened the function can be triggered. The way to do it is very similar to how you bind event on Unreal. Here is an example of one class:

~~~c++
eae6320::AHitTestActor::AHitTestActor(eae6320::Graphics::Mesh* i_mesh, eae6320::Graphics::Effect* i_effect) : GameFramework::AActor(i_mesh, i_effect)
{
    boxComp = new Collision::BoxCollisionComponent();
    boxComp->SetCollisionEvent(Collision::CollisionEvent::Hit);
    boxComp->SetCollisionComponentType(Collision::CollisionComponentType::Static);

    Collision::CollisionManager::GetCollisionManager()->AddCollisionComponent(*boxComp);

    //Bind Event
    boxComp->UpdatePositionAfterCollision.Add(this, &AHitTestActor::UpdatePosition);
    boxComp->OnComponentHit.Add(this, &AHitTestActor::OnComponentHit);
}
~~~



## What's Next

- Incremental update the dynamicBVH to avoid the entire reconstruction to optimal performance;
- Find other code logic that can be optimized and fix any potential issues;
- Attempt to create a visual interface to enhance the user experience



## Collision System Demo

Download and have a try: [MyGame](https://drive.google.com/uc?export=download&id=1PNvfhmmaixJHxYpNe8DRQPGUtY_JG6oA)
