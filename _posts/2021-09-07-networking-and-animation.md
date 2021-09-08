---
layout: post
title: Networking and Animations! 
categories: GameDev Networking Animation
---

This is my first checkpoint and this week I've made some progress on networking implementation and on a basic animation setup.

Here is the code if you want to open the project by yourself:

[Client](https://github.com/vc0sta/BlightDoor/)
[Server](https://github.com/vc0sta/BlightDoor-Server)

## Animations

### Mixamo + Blender + Godot

Regarding to animations, first I got downloaded some random model from [Mixamo](https://www.mixamo.com/) and both animations that I've been willing to use. This model and animations were in **.fbx** format, and despite Godot is compatible with this format, I haven't imported it directly into the engine.

Instead, I used [Blender](https://www.blender.org/) with an awesome plugin called [Godot Game Tools](https://viniguerrero.itch.io/godot-game-tools). This allowed me to join all animations with my model and export it as a single **.gltf** file.

I don't know if it is the best way to integrate all this tools, but workflow was so simple and everything intergrated so well, that I'll be using it more times.

ps: **Godot Game Tools** even has this **Root Motion** feature with some fixes when working with **Mixamo+Godot**, but as I'm working on a multiplayer game now, I skipped it and I'm using simple animations and character movement.

### Animation Tree

With the model and animations already in godot, I've been using the super cool [Animation Tree](https://docs.godotengine.org/en/stable/tutorials/animation/animation_tree.html) feature on Godot, and created a simple blend on 2 animations (Idle and Running).

For those who don't know what I'm talking about, basically I have 2 **Animation** nodes linked in a **Blend** node before going to the **Output** node.

![Animation Tree]({{site.baseurl}}/post_images/2021-09-07/animation-tree.png)

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

![Networking Architecture]({{site.baseurl}}/post_images/2021-09-07/network-architecture.png)

The idea here is that I have a **server**, source of all truth.. and the **clients** that will be rendering the game to the players. Both the server and the client are written in Godot.

Maybe that's not the best idea when talking about performance to have the server as a Godot intance as well, but it's good enough for learning purposes now. I guess the performance can be improoved later by identifying just the most intensive workloads and migrating some of them to a more performatic implementation.

### Basic Communication
https://github.com/vc0sta/BlightDoor/blob/7f1dfe3fc01a357831b6129eecb989a67a7c45ae/Map.gd#L45
I've only created a very primitive **Client x Server** communication here, basically what we have is a **client** sending the current **state** of the Player to the server, and the **server** broadcasting this information to all connected **clients**.

The **player state** being sent over the internet is basically the current **position** of a player, it's **rotation** and the **blend_amount** value to sync the movement animations.

To implement all of this, I've only used Godot's native [high-level](https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html) networking.

### Interpolation and Extrapolation

There are 2 concepts that I've implemented following [this](https://www.youtube.com/watch?v=w2p0ugw3afs&list=PLZ-54sd-DMAKU8Neo5KsVmq8KtoDkfi4s&index=13) and [this](https://www.youtube.com/watch?v=XGyrKmOxLcc&list=PLZ-54sd-DMAKU8Neo5KsVmq8KtoDkfi4s&index=14) tutorials.

The first one is **Interpolation**, that is basically smoothly transitioning from a value to another one. This is used not only in multiplayer games, but what I want to write here is specifically about using **interpolation** to smoothly change the position of a Player in a multiplayer game.

I'll not enter at code level here, but the basic "algorythim" behind this concept is creating a **Buffer** to store some **Player states** and then smoothly transitioning from one **state** to the next.. This will cause an effect of continuity, and you will not see other players teleporting from a position to another because the **client** is filling the gaps between them.

**Extrapolation** is quite the same idea, but it tries to mitigate network errors (everyone knows that networking are not reliable.. right?) by creating future **states** based on the one we already have. And as the time goes by and we start receiving new **states** again, the **interpolation** between what was foreseen and the actual **state** of a given player will make everything adjusted again..

If you want to see the full implementation, here is the code for [interpolation](https://github.com/vc0sta/BlightDoor/blob/7f1dfe3fc01a357831b6129eecb989a67a7c45ae/Map.gd#L29) and [extrapolation](https://github.com/vc0sta/BlightDoor/blob/7f1dfe3fc01a357831b6129eecb989a67a7c45ae/Map.gd#L45) (further explanations on how it was programmed can be found on [Game Development Center's channel](https://www.youtube.com/watch?v=w2p0ugw3afs&list=PLZ-54sd-DMAKU8Neo5KsVmq8KtoDkfi4s&index=13)).
