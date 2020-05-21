---
layout: post
title: Making a roguelike in C# using GoRogue and SadConsole (Part One)
header-img: "img/posts/runic-quest-part-one-final.jpg"
tags: [tutorials, programming projects, c#] 
---

After having completed several projects using Swift over the past four months (my latest and most ambitious being [Linnstrument Helper](https://markjames.dev/linnstrument-helper), I started to look for other projects which would pique my interest and improve my coding skills. Earlier this week, I was speaking to my brother who is currently playing an aRPG known as Grim Dawn. I started thinking back to my old days of playing [Dungeon Crawl Stone Soup](https://crawl.develz.org/) on an underpowered laptop as a student and decided to try creating my own simplistic roguelike. 

As my programming projects have increased in complexity (I've now written both console and GUI apps for MacOS and iOS), I have discovered that the more projects you complete, the better you get at being able to roughly estimate how much work might be involved in a project. With so many different things to keep track of (even for a simple roguelike), I knew that this project would be my biggest undertaking so far. I don't expect to create a roguelike with the depth of something like DCSS or [Nethack](https://www.nethack.org/), but I have set a medium-term goal of creating something which would hold the interest of both my brothers for at least thirty minutes when I eventually have them try it out. As I find that I learn better while writing about a subject, this series is intended as being a build log/tutorial of my experience. By the end of Part One of this tutorial, you should have a basic game which looks something like the following:

![Runic Quest tutorial part one build](https://markjames.dev/img/posts/runic-quest-build-one.jpg "Screenshot of the end of Part One")

## Cross-Platform Considerations

My initial impulse was to use Swift to create the game (since I've been using it extensively for the past several months), but it soon became apparent that Swift wouldn't be a great candidate for a roguelike. Firstly, cross-platform support is lacking at the moment, and I wanted to be able to create a game which I could share with my brothers who use Windows. Secondly, I wanted to make use of some libraries to offload the heavy lifting tasks (such as map generation and pathfinding) while I focused on other areas of the game. After more research, I finally settled on using C#. Firstly, thanks to [.NET Core](https://dotnet.microsoft.com/download), I had access to a powerful and mature cross-platform framework to build my game. Secondly, C# has two great libraries which are conducive to creating great roguelikes - [Go Rogue](https://github.com/Chris3606/GoRogue) and [Sadconsole](https://sadconsole.com/). While Go Rogue is a .NET Standard roguelike library written in C#, SadConsole is a "MonoGame 3.7-based game library that provides an engine to emulate old-school console and command prompt style graphics". There's also a great helper library known as [Sadconsole.GoRogueHelpers](https://github.com/thesadrogue/SadConsole.GoRogueHelpers) which bridges the gaps between the two.

I have a bit of experience with C#, and its similarities to Swift made the transition to a the language a breeze. As Swift is still rather young, the tooling just isn't quite at the same level of the .net ecosystem, and I found working with NuGet to be an awesome experience!

## Recommended Reading

Fortunately, I was able to find a superb tutorial series which was able to take me quite far into my journey. Aptly named [The SadConsole Roguelike Tutorial Series](https://ansiware.com/), the guide explains the basics of using Sad Console to create a roguelike. Getting started, I'm going to assume that you've read and followed that guide up until part 13 as it explains a lot about the roguelike creation process (although it is optional since we'll be working with a NuGet template soon!). 

In past projects, I've used Git in a superficial manner by relying on the XCode Git interface and pushing all my commits directly to the master branch. This time, I plan to Create new branches for each feature I'm working on, and then merge those with the master. As I plan to contribute to larger open source projects in the future, I'd like to reinforce good Git habits sooner than later!

## Setting up the SadConSole.GoRogueHelpers Template

On Nuget, there's a [handy template](https://www.nuget.org/packages/TheSadRogue.GoRogueSadConsole.Templates/)  Created by Chris3606 (Go Rogue's creator) which starts you off with a simple game template which supports:
* Map generation
* Player movement
* Static enemy spawning
* FOV rendering

Unlike the Ansiware tutorial, this template uses the SadConsole Go Rogue integration library in order to unlock Go Rogue's powerful features such as map generation and line of sight.

After some helpful chats with Thraka and Chris3606, (The respective creators of SadConsole and Go Rogue), I was pointed to a [NuGet Template](https://www.nuget.org/packages/TheSadRogue.GoRogueSadConsole.Templates/) which I could use to build a more complete project.

To install the template, open a new command prompt and run:

```
dotnet new --install TheSadRogue.GoRogueSadConsole.Templates
```

Next, in your project folder, run 
```
dotnet new goroguesadconsolegame -o some_new_folder
```

After running the commands, I was left with a folder which contained the following files:

* MapScreen.cs
* Player.cs 
* Program.cs
* RunicQuest.csproj
* RunicQuestMap.cs

Running the template project produced this game window:

![Running the GoRogue template build](https://markjames.dev/img/posts/gorogue-template-build.jpg "Screenshot of the GoRogue template")

Awesome, we now have a base template to build off of!

## Adjusting the Map size

The first thing I wanted to implement was a center on player feature. In order to do so, I needed to create a map larger than the viewport.  

I edited Program.cs and modified the starting constants in order to separate the map and viewport dimensions. At the top of the Program class, I added:

```csharp
private const int ViewPortWidth = 80;
private const int ViewPortHeight = 25;

private const int MapWidth = 500;
private const int MapHeight = 500;
```

Now we have two sets of definitions, one for the game map size, and the other for the view size.

Next, I modified the game creation constructor to accept the viewport variables:

```csharp
// Setup the engine and create the main window.
SadConsole.Game.Create(ViewPortWidth, ViewPortHeight);
```

Lastly, I modified the MapScreen constructor arguments so that we're passing both the game size and viewport size parameters:

```csharp
MapScreen = new MapScreen(MapWidth, MapHeight, ViewPortWidth, ViewPortHeight);
```

After compiling the game, I was now greeted with the following screen:

![Runic Quest initial commit screenshot](https://markjames.dev/img/posts/runic-quest-center-map.jpg "Runic Quest screenshot")

Now that the map size is larger than the viewport, the player remains centered on the map as we explore it!

After adjusting the map and viewport sizes, its a good idea to modify the GenerateDungeonMazeMap() argument parameters inside MapScreen.cs in order to be more suitable for a 500x500 map. I'm not sure what the ideal parameters are for a room of this size is, and so I recommend playing around until you find something that works for you. For me, the following settings worked well for now:

```csharp
QuickGenerators.GenerateDungeonMazeMap(tempMap, minRooms: 800, maxRooms: 140, roomMinSize: 12, roomMaxSize: 24);
```

## Change the player and wall colors

The following two changes were simple to make, and also give the game a bit of character. In the Player.cs file, I changed the Player() constructor color like so:

```csharp
public Player(Coord position) : base(Color.LightGreen, Color.Black, '@', position, (int)MapLayer.PLAYER, isWalkable: false, isTransparent: true) => FOVRadius = 10;
```
Aside from changing the colors, you can also change the player symbol, FOV radius, and clipping layers.

For the curious, you can see (and fork!) my entire codebase [here on Github](https://github.com/markjamesm/runic-quest).

## Part One Conclusion

In part one of the article, we did a whole lot of stuff that may take several days to digest. Firstly, we installed the GoRogue integration template and then laid the groundwork for our own game by changing the map size, centering the player, and changing a few colors.

If at any time you find yourself stuck, I suggest leaving a comment below and I'll do my best to help answer! Moreover, there's a terrific community on the [SadConsole Discord Server](https://discord.gg/TagZMGg) that I highly recommend checking out. The devs of both Sadconsole and Go Rogue are very active (and super helpful!) there, and there's also a friendly dev community with other members eager to help out.

Stay tuned for the upcoming part two where we'll implement even more features to Runic Quest!