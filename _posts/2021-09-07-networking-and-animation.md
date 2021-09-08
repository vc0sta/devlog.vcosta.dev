---
layout: post
title: Networking and Animations! 
categories: GameDev
---

This is my first checkpoint and this week I've made some progress on networking implementation and on a basic animation setup.

## Animations

### Mixamo + Blender + Godot

Regarding to animations, first I got downloaded some random model from [Mixamo](https://www.mixamo.com/) and both animations that I've been willing to use. This model and animations were in **.fbx** format, and despite Godot is compatible with this format, I haven't imported it directly into the engine.

Instead, I used [Blender](https://www.blender.org/) with an awesome plugin called [Godot Game Tools](https://viniguerrero.itch.io/godot-game-tools). This allowed me to join all animations with my model and export it as a single **.gltf** file.

I don't know if it is the best way to integrate all this tools, but workflow was so simple and everything intergrated so well, that I'll be using it more times.

ps: **Godot Game Tools** even has this **Root Motion** feature with some fixes when working with **Mixamo+Godot**, but as I'm working on a multiplayer game now, I skipped it and I'm using simple animations and character movement.

### Animation Tree

With the model and animations already in godot, I've been using the super cool [Animation Tree](https://docs.godotengine.org/en/stable/tutorials/animation/animation_tree.html) feature on Godot, and created a simple blend on 2 animations (Idle and Running).

For those who don't know what I'm talking about, basically I have 2 **Animation** nodes linked in a **Blend** node before going to the **Output** node.

The **Output** node is the last node at the pipeline, and everything that comes into it, will be used to animate the model.

The **Animation** node represents each animation I have on my **AnimationPlayer** (which was created automatically by **GGT**).

And last but not least, the **Blend** node is responsible for mixing (blending) both animations based on a value ranging from **0** to **1**, so if this node has the **blend_amount** set at **0**, the animation playing is **Idle**, if we have this value set to **1**, the animation will be **Running**, and if there is any other value between these two, the animation will be blended somewhere between **Idle** and **Running**.

For example:

| 0    | Idle           |
| 0.25 | Walking Slow   |
| 0.5  | Walking        |
| 0.75 | Walking Fast   |
| 1    | Running        |

So, I dont need to create a very complex animation controller to deal with all possibilities here, I actually only need to take care of **1** value, and Godot will properly animate this model for me.


## Networking

Well, this will be more difficult to explain ~~because I only copied it from the tutorials~~...

So.. lets begin with a simple diagram:

![Networking Architecture]({{site.baseurl}}/images/2021-09-07/network-architecture.png)

The idea here is that I have a **server**, source of all truth.. and the **clients** that will be rendering the game to the players.:w

