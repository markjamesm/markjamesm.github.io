---
layout: post
title: Making a roguelike in C# using GoRogue and SadConsole (Part Two)
header-img: "img/posts/runic-quest-part-one-final.jpg"
tags: [tutorials, programming projects, c#] 
---

After setting up the GoRogue Helpers template and making a few basic changes to it in [Part One](https://markjames.dev/2020-05-21-making-a-roguelike-in-c-with-gorogue-sadconsole-part-one/) of this tutorial series, Part Two looks to dig into the code even deeper and implement some additional features to make our game feel more like a game. Assuming that you completed part one, you should have a basic game which compiles and lets you explore a fairly big dungeon map which contains static enemies which you can't attack. 

## Creating a UI Manager

The first thing that I wanted to do going forward, was to implement a proper UI manager. Fortunately, [Part 8](https://ansiware.com/tutorial-part-8-user-interface-manager-v8/) of the Ansiware tutorial explains how to implement a UI manager. 

The first thing I did was to create a class called UIManager.cs, and I copied and pasted the Ansiware code to use as a blueprint, modifying it for my game name:

```csharp
using Microsoft.Xna.Framework;
using SadConsole;
namespace RunicQuest
{
    // Creates/Holds/Destroys all consoles used in the game
    // and makes consoles easily addressable from a central place.
    public class UIManager : ContainerConsole
    {
        public ScrollingConsole MapConsole;

        public UIManager()
        {
            // must be set to true
            // or will not call each child's Draw method
            IsVisible = true;
            IsFocused = true;

            // The UIManager becomes the only
            // screen that SadConsole processes
            Parent = SadConsole.Global.CurrentScreen;
        }

        // Creates all child consoles to be managed
        // make sure they are added as children
        // so they are updated and drawn
        public void CreateConsoles()
        {
            
        }

    }
}
```

Here we have a lot of great base code! The next step is to hook in the UI Manager to our MapScreen class. Currently, the MapScreen.cs file contains the following line of code:

```csharp
internal class MapScreen : ContainerConsole
```

Since our UIManager extends the ContainerConsole base class, I wired it up to UIManager instead:

```csharp
internal class MapScreen : UIManager
```

Keeping in line with Ansiware's theme of encapsulation, the next step was to move the map generation code outside of GameLoop.cs (previously named Program.cs, but I renamed it for readability) and into the UIManager. In order to do so, I removed the references to the Map Height and Width, as well as the details and added the followingt code inside UIManager.cs:

```csharp
    public class UIManager : ContainerConsole
    {
        // Set the map and viewport dimensions.
        public const int ViewPortWidth = 80;
        public const int ViewPortHeight = 25;

        private const int MapWidth = 500;
        private const int MapHeight = 500;
```

Note that the ViewPort height and width are set to a public constant, this is because we still need to use them inside our GameLoop.cs to create the main window: 

```csharp
        private static void Main()
        {
            // Setup the engine and create the main window.
            SadConsole.Game.Create(UIManager.ViewPortWidth, UIManager.ViewPortHeight);
```

Now we need to create the Map inside of our UIManager class, and we do so inside of the CreateConsoles() method like this:

```csharp
        public void CreateConsoles()
        {
            // Generate and display the map
            MapScreen = new MapScreen(MapWidth, MapHeight, ViewPortWidth, ViewPortHeight);
        }
```

CreateConsoles() creates all the child consoles to be managed, and makes sure that they are added as children, updated, and drawn.

Now we need to wire up the UIManager to our GameLoop. Firstly, we declare our UIManager:

```csharp
    internal class GameLoop
    {

        public static UIManager UIManager;
```

and then hook in UIManager inside our init method:

```csharp
            // Create our UI Manager and then spawn our consoles.
            UIManager = new UIManager();
            UIManager.CreateConsoles();
```

At this point, if you save and run the code you should see the following window appear:

![Runic Quest UI Manager](https://markjames.dev/img/posts/runic-quest/runic-quest-ui-manager.jpg "Runic Quest roguelike using a UI Manager")

Success! The GoRogue template map generator is now hooked into our freshly minted UIManager class! Although this is a great first step, now is a good time to make an attempt at hooking in a basic message log. Again, I'm going to be borowing code from Ansiware, this time from [Part 9](https://ansiware.com/tutorial-part-9-message-log-v8/)

Creating MessageLogWindow.cs, I filled it with the following code:

```csharp
using System.Collections.Generic;
using SadConsole;
using System;
using Microsoft.Xna.Framework;

namespace RunicQuest
{
    //A scrollable window that displays messages
    //using a FIFO (first-in-first-out) Queue data structure
    public class MessageLogWindow : Window
    {
        //max number of lines to store in message log
        private static readonly int _maxLines = 50;

        // a Queue works using a FIFO structure, where the first line added
        // is the first line removed when we exceed the max number of lines
        private readonly Queue<string> _lines;

        // the messageConsole displays the active messages
        private SadConsole.ScrollingConsole _messageConsole;

        // Create a new window with the title centered
        // the window is draggable by default
        public MessageLogWindow(int width, int height, string title) : base(width, height)
        {
            // Ensure that the window background is the correct colour
        //    Theme.WindowTheme.FillStyle.Background = DefaultBackground;
            _lines = new Queue<string>();
            CanDrag = true;
            Title = title.Align(HorizontalAlignment.Center, Width);

            // add the message console, reposition, and add it to the window
            _messageConsole = new SadConsole.ScrollingConsole(width - 1, height - 1);
            _messageConsole.Position = new Point(1, 1);
            Children.Add(_messageConsole);
        }

        //Remember to draw the window!
        public override void Draw(TimeSpan drawTime)
        {
            base.Draw(drawTime);
        }

        //add a line to the queue of messages
        public void Add(string message)
        {
            _lines.Enqueue(message);
            // when exceeding the max number of lines remove the oldest one
            if (_lines.Count > _maxLines)
            {
                _lines.Dequeue();
            }
            // Move the cursor to the last line and print the message.
            _messageConsole.Cursor.Position = new Point(1, _lines.Count);
            _messageConsole.Cursor.Print(message + "\n");
        }
    }
}
```

Next, I declared my MessageLogWindow inside my UIManager class, and then added the following code to the CreateConsoles() method just below the other code:

            // Create the message log window and set its position.
            MessageLog = new MessageLogWindow(ViewPortWidth, ViewPortHeight / 4, "Message Log");
            Children.Add(MessageLog);
            MessageLog.Show();
            MessageLog.Position = new Point(0, 20);

            // Print a test method
            MessageLog.Add("Testing 123");
            
Running my code at this point produces the following result:

![Runic Quest MessageLogger](https://markjames.dev/img/posts/runic-quest/runic-quest-message-log-window.jpg "Runic Quest roguelike with message log")



