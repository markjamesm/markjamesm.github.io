---
layout: guide
title: Developing a Lightweight TUI Music Player in C# using Terminal.Gui (Part Four)
header-img: "img/posts/music-sharp/MusicSharp-playback-controls.png"
tags: [guides, programming projects, c#, MusicSharp] 
---

In <a href="/2020-11-06-developing-tui-music-player-csharp-part-three/" target=_blank>Part Three</a> of this guide, we implemented Dependency Injection (DI) through the use of constructor injection and expanded our player interface by adding volume controls. With all these features in place, the final part of this guide will add some more key features including loading playlist. At the end of this tutorial, our player should look something like this:

<img src="/img/posts/music-sharp/musicsharp-tui.png" width="750" height="402" alt="MusicSharp build with audio streaming support">

I also recommend taking a look at <a href="https://github.com/markjamesm/MusicSharp" target=_blank>MusicSharp on Github</a> where you can clone a working version to follow along with.

## Loading Audio Playlists

One of the key features of a music player is being able to load audio playlists, and so I wanted to make sure to implement this feature before a 1.0 release. I did some research and managed to find a great playlist library known as <a href="https://github.com/Zeugma440/atldotnet" target=_blank>atldotnet</a> which supports a multitude of formats, but for now I'll just be working with M3U since its ubiquitous. I decided to create a PlayListLoader class and place it inside my model folder as PlaylistLoader.cs. Inside the PlayListLoader class, I created a method which takes an M3U playlist filepath and returns the filepaths of the tracks inside it.  

```csharp
/// <summary>
/// The PlaylistLoader class loads a playlist of a given type.
/// </summary>
public class PlaylistLoader
{
    // This will be used in the future to allow for playlist types beyond M3U.
    // public virtual void LoadPlaylist() { }

    /// <summary>
    /// Load an M3U playlist.
    /// </summary>
    /// <returns>Returns a list of playlist information.</returns>
    /// <param name="userPlaylist">The user specified playlist path.</param>
    public List<string> LoadPlaylist(string userPlaylist)
    {
        IPlaylistIO theReader = PlaylistIOFactory.GetInstance().GetPlaylistIO(userPlaylist);

         List<string> filePaths = new List<string>();

        foreach (string s in theReader.FilePaths)
        {
            filePaths.Add(s);
        }

        return filePaths;
    }
}
```

With this code in place, we can now create a FileOpen dialog inside Tui.cs which allows only M3U files:

```csharp
// Load a playlist file. Currently, only M3U is supported.
private void LoadPlaylist()
{
    var d = new OpenDialog("Open", "Open a playlist") { AllowsMultipleSelection = false };

    // This will filter the dialog on basis of the allowed file types in the array.
    d.AllowedFileTypes = new string[] { ".m3u" };
    Application.Run(d);

    if (!d.Canceled)
    {
        this.playlist = this.playlistLoader.LoadPlaylist(d.FilePath.ToString());

        if (this.playlist == null)
        {
            Application.RequestStop();
        }
        else
        {
            foreach (string track in this.playlist)
            {
                playlistTracks.Add(track);
            }

            Application.Run();
        }
    }
}
```

Inside the nested if statement, we check to see if the playlist is null, and if not we call our LoadPlaylist() method. We then need to create a new list to hold the playlist tracks, as well as a view for it:

```csharp

// Create the left-hand playlists view.
playlistPane = new FrameView("Playlist Tracks")
{
    X = 0,
    Y = 1, // for menu
    Width = Dim.Fill(),
    Height = 23,
    CanFocus = false,
};

// The list of tracks in the playlist.
playlistTracks = new List<string>();

playlistView = new ListView(playlistTracks)
{
    X = 0,
    Y = 0,
    Width = Dim.Fill(),
    Height = 23,
    AllowsMarking = false,
    CanFocus = true,
};
```

Finally, we use a lambda to play a selection whenever the user clicks on a playlist entry: 

```csharp
// Play the selection when a playlist path is clicked.
playlistView.OpenSelectedItem += (a) =>
{
    this.player.LastFileOpened = a.Value.ToString();
    this.player.PlayFromPlaylist(this.player.LastFileOpened);
    this.NowPlaying(this.player.LastFileOpened);
    this.TrackLength();
    this.TimePlayed();
};

playlistPane.Add(playlistView);
```

After testing this out, everything works as expected. Success!

## Implementing Streaming

Implementing audio streaming was fairly easy thanks to the NAudio library. Inside IPlayer.cs, I added the following method to setup audio playing:

```csharp
/// <summary>
/// Method to play an audio stream from a URL.
/// </summary>
/// <param name="streamURL">The stream URL of the audio file to play.</param>
void OpenStream(string streamURL);
```

In the future, I may refactor this out into an IStreamable interface to separate streaming from the rest of the audio player, making it easier to implement barebones cross platform support by starting only with the local audio playing capabilities.

The next step was to implement the interface method inside the WinPlayer class:

```csharp
/// <summary>
/// Method to open an audio stream.
/// </summary>
/// <param name="streamURL">The URL of the stream.</param>
public void OpenStream(string streamURL)
{
    try
    {
        using (var mf = new MediaFoundationReader(streamURL))
        {
            this.outputDevice.Init(mf);
            this.outputDevice.Play();
        }
    }
    catch (System.ArgumentException)
    {
    }
    catch (System.IO.FileNotFoundException)
    {
    }
}
```

Lastly, we'll create a dialog to open the stream

```csharp
// Open and play an audio stream.
private void OpenStream()
{
    var d = new Dialog("Open Stream", 50, 15);

    var editLabel = new Label("Enter the url of the audio stream to load:\n(.mp3 only)")
    {
        X = 0,
        Y = 0,
        Width = Dim.Fill(),
    };

    var streamURL = new TextField(string.Empty)
    {
        X = 3,
        Y = 4,
        Width = 42,
    };

    var loadStream = new Button(12, 7, "Load Stream");
    loadStream.Clicked += () =>
    {
        this.player.OpenStream(streamURL.Text.ToString());
        Application.RequestStop();
    };

    var cancelStream = new Button(29, 7, "Cancel");
    cancelStream.Clicked += () =>
    {
       Application.RequestStop();
    };

    d.AddButton(loadStream);
    d.AddButton(cancelStream);
    d.Add(editLabel, streamURL);
    Application.Run(d);
}
```
At the moment, streaming support works with MP3 (and a select few other formats) and so I tested the player using a direct streaming link from Soma.fm:

<img src="/img/posts/music-sharp/musicsharp-open-stream.png" width="750" height="402" alt="MusicSharp build with audio streaming support">

Loading the MP3 worked just as well as a local file!

## Building the Now Playing section

The third and final step we'll be working on in this series is implementing some visual information regarding playback state through the use of a progressbar and some labels. To start, I added two new methods to IPlayer:

```csharp
/// <summary>
/// Returns the current playtime of the audioFileReader instance.
/// </summary>
/// <returns>The current time played as TimeSpan.</returns>
System.TimeSpan CurrentTime();

/// <summary>
/// Returns the total track length in timespan format.
/// </summary>
/// <returns>The length of the track in timespan format.</returns>
System.TimeSpan TrackLength();
```

The above methods will allow us to get the current time played of a track as well as its length. Now, inside WinPlayer.cs we can implement these methods:

```csharp
/// <inheritdoc/>
public System.TimeSpan CurrentTime()
{
    TimeSpan zeroTime = new TimeSpan(0);

    if (this.outputDevice.PlaybackState != PlaybackState.Stopped && this.outputDevice.PlaybackState != PlaybackState.Paused)
    {
        return this.audioFileReader.CurrentTime;
    }
    else
    {
        return zeroTime;
     }
}

/// <inheritdoc/>
    public System.TimeSpan TrackLength()
    {
         return this.audioFileReader.TotalTime;
    }
```
Whereas TrackLength() essentially acts as a read only property for the totaltime of the audio file reader, CurrentTime() checks whether or not the playback is paused or stopped. Reviewing the code now, I could likely replace the if statement with a shorter version using (this.outputDevice.PlaybackState == PlaybackState.Playing).

Lastly, inside Tui.cs we create the playback items:

```csharp
private object mainLoopTimeout = null;
// Define a tick as being one second
private uint mainLooopTimeoutTick = 1000; // ms

internal ProgressBar AudioProgressBar { get; private set; }

// Create the audio progress bar
this.AudioProgressBar = new ProgressBar()
{
    X = 0,
    Y = 2,
    Width = Dim.Fill() - 15,
    Height = 1,
    ColorScheme = Colors.Base,
};

nowPlaying.Add(this.AudioProgressBar);

private void NowPlaying(string track)
    {
        trackName = new Label(track)
        {
            X = 0,
            Y = 0,
            Width = Dim.Fill(),
        };

        nowPlaying.Add(trackName);
    }

private void TrackLength()
{
    // Show the track length in mm:ss format
    trackLength = new Label(this.player.TrackLength().ToString(@"mm\:ss"))
    {
        X = Pos.Right(this.AudioProgressBar) + 7,
        Y = 2,
    };

     nowPlaying.Add(trackLength);
}

private void TimePlayed()
{
    this.AudioProgressBar.Fraction = 0F;

    // Get the duration of the playing track in seconds.
    double counter = Convert.ToInt32(this.player.TrackLength().TotalSeconds);

    // Add the one-second tick to the Terminal.UI mainloop
    this.mainLoopTimeout = Application.MainLoop.AddTimeout(TimeSpan.FromMilliseconds(this.mainLooopTimeoutTick), (loop) =>
    {
        // Increment timer as long as the playtime isn't at zero.
        while (counter != 0 && this.player.IsAudioPlaying)
        {
            // 1 represets 100% progress, so increment by 1 divided by tracklength to get an even update tick.
            this.AudioProgressBar.Fraction += (float)(1 / this.player.TrackLength().TotalSeconds);
            this.TimePlayedLabel(this.player.CurrentTime().ToString(@"mm\:ss"));
            counter--;
            return true;
        }

        return false;
        });
    }

// Update the label with time played.
private void TimePlayedLabel(string timePlayed)
{
    trackName = new Label($"{timePlayed} / ")
    {
        X = Pos.Right(this.AudioProgressBar),
        Y = 2,
        };

        nowPlaying.Add(trackName);
    }
}
```

The code above is fairly lengthy, but I've added comments to make it clear what everything is doing. After compiling the code, I was left with something similar to this (minus the currently non-functional seek buttons):

<img src="/img/posts/music-sharp/musicsharp-tui.png" width="750" height="402" alt="Final state of MusicSharp">

You can now see the progress bar representing the proper amount, as well as the playtime being displayed! Our player is now starting to look and feel a lot more like an audio player! However, I did notice that the progress bar mainloop code runs into timing issues if you try to play a second file in a playlist. I suspect that this may be due to the way I setup the loop, and in the future I would possibly add code to stop and reset the progressbar properly, but for now it works well enough for single files.  

Now that we have a basic player in place, I'm going to end my series here as I consider the project to be mostly complete for my learning purposes. Working on MusicSharp over the past month has given me some great practice coding to interfaces, using Dependency Injection, and getting hands-on experience unit tests by MSTest to test much of the codebase. Much work is still to be done however, and I encourage you to <a href="https://github.com/markjamesm/MusicSharp" target=_blank>fork the project on Github</a>, add a feature or bugfix, and submit a PR! In the coming months, I plan to continue to work on improvements to MusicSharp, but at a much slower pace as I'm currently in the planning stages of a new project!