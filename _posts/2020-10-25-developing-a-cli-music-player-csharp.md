---
layout: guide
title: Developing a Lightweight TUI Music Player in C# using Terminal.Gui (Part One)
header-img: "img/posts/music-sharp/musicsharp-open-dialog.jpg"
tags: [guides, programming projects, c#, MusicSharp] 
---

Recently, I've been listening to a lot of <a href="https://somafm.com" target="_blank">SomaFM</a> internet radio streams while I work as they have a lot of terrific commercial-free programming. One day while being creatively inspired by the Sonic Universe station's offerings, I had the idea of creating my own music player with support for streaming. Not only would this be a great project for continuing to learn C# and Dotnet Core, but it would also allow me to build a simple, lightweight player customized to my liking! Note: you can find parts two and three of this guide here:

<a href="/2020-10-29-developing-tui-music-player-csharp-part-two/">Part Two</a> | <a href="/2020-11-06-developing-tui-music-player-csharp-part-three/">Part Three</a>

After a brainstorming session, I came up with the name MusicSharp and a core list of features I planned to implement:

- Play audio files and streams
- Save and load playlists
- Be lightweight
- Be CLI-based
- Cross-platform support

Moreover, I also wanted to make sure that I made use of Test Driven Development and code linting within the project. For this project, I'm making use of Stylecop to enfore proper code style. For the curious, you can view the entire MusicSharp code repository in its current state <a href="https://github.com/markjamesm/MusicSharp" target="_blank">on Github</a>. I highly recommend following the repo to keep up to date with the latest commits (and a star wouldn't be too bad either :-P)!

## Getting Started

Armed with a name and an idea, the next thing I needed was to settle on was which C# libraries I would use to implement the UI and audio functionality. After doing some research, I decided that the optimal solution would be to use <a href="https://github.com/migueldeicaza/gui.cs" target="_blank">Terminal.Gui</a> for the UI elements and <a href="https://github.com/naudio/NAudio" target="_blank">NAudio</a> to handle the audio side of things. Created by Miuel de Icaza (the creator of Mono and Xamarin), Terminal.Gui is a cross-platform library for building console-based applications that work on monochrome terminals as well as modern terminals with mouse support. Leveraging Termnial.Gui to build the UI, I chose NAudio to handle the audio playing capabilities of MusicSharp as its a mature C# library.

## Mocking up a simple UI

With some of the preliminary items out of the way, my next step was to work on some wireframe mockups to see what kind of UI I might want to create. I've used a lot of music players over my life (including CLI ones), and so I have a good idea of what music players should have. Slowly, I started breaking down the different elements I thought would need and came up with the following:

- A menu bar
- A status bar
- A player section (with associated controls)
- A playlist browser

I sketched out a few different UI layouts until I came up with the following which I felt would work for a first iteration:

<img src="/img/posts/music-sharp/MusicSharp-ui-mockup.png" width="366" height="302" alt="MusicSharp UI mockup">

## Building the Main Window

Mockup in hand, I started diving into the Terminal.gui API docs in order to build my first window. I created a new Gui class in a file called Gui.cs with a method named Start(). Inside the Start method, I initialized Terminal.Gui and created a simple top-level window. Note that 

```csharp
namespace MusicSharp
{
    using Terminal.Gui;

    /// <summary>
    /// The Gui class houses the CLI elements of MusicSharp.
    /// </summary>
    public class Gui
    {
        /// <summary>
        /// The Start method builds the user interface.
        /// </summary>
        public void Start()
        {
            // Creates a instance of MainLoop to process input events, handle timers and other sources of data.
            Application.Init();

            var top = Application.Top;
            var tframe = top.Frame;

            // Create the top-level window.
            var win = new Window("MusicSharp")
            {
                X = 0,
                Y = 1, // Leave one row for the toplevel menu

                // By using Dim.Fill(), it will automatically resize without manual intervention
                Width = Dim.Fill(),

                // Subtract one row for the statusbar
                Height = Dim.Fill() - 1,
            };
            });

            // Add the layout elements and run the app.
            top.Add(win);
            Application.Run();
        }
    }
}
```

Inside my Program.cs file, I called the Start method like so:

```csharp
    public static void Main()
    {
        // Start MusicSharp.
        Gui gui = new Gui();
        gui.Start();
    }
```

Running the program in this state produced the following result:

<img src="/img/posts/music-sharp/MusicSharp-first-build.png" height="450" width="266" alt="MusicSharp first build screenshot">

Success! We now have a top level window which dynamically resizes while leaving space on the top and bottom of the screen for statusbars. 

## Creating a basic menu

The next step was to implement a top level menu, and fortunately this was easy to do with Terminal.Gui. Just aboce top.Add(win), I created the following code:

```csharp
// Create the menubar.
var menu = new MenuBar(new MenuBarItem[]
{
    new MenuBarItem("_File", new MenuItem[]
    {
        new MenuItem("_Open", "Open a music file", () => Application.RequestStop()),

        new MenuItem("_Open Stream", "Open a stream", () => Application.RequestStop()),

        new MenuItem("_Quit", "Exit MusicSharp", () => Application.RequestStop()),
    }),

    new MenuBarItem("_Help", new MenuItem[]
    {
        new MenuItem("_About", string.Empty, () =>
        {
            MessageBox.Query("Music Sharp 0.2.0", "\nMusic Sharp is a lightweight CLI\n music player written in C#.\n\nDeveloped by Mark-James McDougall\nand licensed under the GPL v3.\n ", "Close");
        }),
    }),
});
```

Here, I did a few things. Firstly, I created a new MenuBar which will house the Menu Bar Items. Next, I created each menu item (File and Help), and populated them with handy child items that we'll need in our UI. These children take a name, optional descriptor, and a method as arguments, and for now I'm calling Application.RequestStop() on each of them to exit the application (Later I'll be replacing these with the actual methods once I write them). In addition, I also created a simple about dialog box as a way to familiarize myself with Terminal.Gui. Compiling our code up to this point produces the following menu:  

<img src="/img/posts/music-sharp/MusicSharp-menu.png" height="450" width="266" alt="Screenshot of the MusicSharp menu bar">

And a neat looking about dialog:

<img src="/img/posts/music-sharp/MusicSharp-about-dialog.png" height="450" width="266" alt="Screenshot of the MusicSharp about dialog">

Although I still need to add some additional menu items (such as save/load playlist functionality), this gives us a great jumping off point for the next part, getting basic audio functionality working! 

## Playing Audio

Now that a very simple UI had been created, my next order of business was to test out NAudio and see how I could play an MP3 file as a simple proof of concept. In order to do so, I created a new file called Player.cs to house the audio playing related functions.

Inside the Player class, I created a PlayAudioFile() method which read an MP3 file from a static location on my PC and then played it using NAudio. The code looked like this:

```csharp
/// <summary>
/// The Player class handles audio playback.
/// </summary>
public static class Player
{
    /// <summary>
    /// Method that implements audio playback from a file.
    /// </summary>
    public static void PlayAudioFile()
    {
        string file = @"C:\MusicSharp\Zhund-Dusty.mp3";

        // Load the audio file and select an output device.
        using var audioFile = new AudioFileReader(file);
        using var outputDevice = new WaveOutEvent();

        outputDevice.Init(audioFile);
        outputDevice.Play();

        // Sleep until playback is finished.
        while (outputDevice.PlaybackState == PlaybackState.Playing)
        {
            Thread.Sleep(1000);
        }
    }
}
```

The last order of business was to wire up the PlayAudioFile() method to the UI, and I did so by replacing the Open menu line of code in Gui.cs to the following:

```csharp
new MenuItem("_Open", "Open a music file", () => Player.PlayAudioFile()),
```

After compiling and opening the player, I was now greeted with the sweet sounds of music! Although it was exciting to hear music this early on, there were quite a few catches however. These included:

- No exception handling if the MP3 file is missing
- No way to load other music files
- Missing playback controls and volume
- Playback status indicator

Although these features were missing, I still find it quite exciting whenever I'm able to make tangible progress in a project and it provides some motivation to work through some of the more tedious parts.

# Closing thoughts

In Part One of this guide, we learned how to implement a simple UI using Terminal.Gui and then use it to play an audio file with NAudio. In later parts of this guide, we will learn how to use Dependency Injection in order to loosely couple the GUI from the player interface, paving the way for platform-specific audio players!. Interestingly, even at this stage I had no problems compiling MusicSharp on my Macbook Pro (running Catalina), except that the program crashed once I tried to open an MP3 (to be expected):

<img src="/img/posts/music-sharp/MusicSharp-macOS-Catalina.png" height="450" width="266" alt="MusicSharp first build running on macOS Catalina">

In these early stages of testing, MusicSharp is currently consuming 15mb of memory and negligible CPU usage while running in debug mode, and it'll be interesting to see where the performance numbers end up as the player functionality gets more fleshed out. Be sure to check out <a href="https://github.com/naudio/NAudio/blob/master/Docs/PlayAudioFileWinForms.md" target=_blank>part two of this guide</a>, as we will refactor our current code and then implement some playback controls!