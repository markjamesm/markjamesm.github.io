---
layout: guide
title: Writing a Linnstrument Companion App with SwiftUI and AudioKit (Part 2)
header-img: "img/posts/linnstrument-helper-og.jpg"
tags: [swift, programming projects, guides, linnstrument helper]
---

Welcome to part two of my Linnstrument Helper build guide! If you're looking for part one, you can [find it here](https://markjames.dev/2020-05-06-writing-a-midi-controller-app-part-one/). I also recommend checking out the project's [Github repository](https://github.com/markjamesm/linnstrument-helper) to follow along with the code examples.

In part two of this series, I discuss how I was able to highlight pressed notes on the Linnstrument grid by using Anchor Preferences. For those who haven't heard of it, Anchor Preferences is a way to pass layout data such as coordinates and element bounds to a parent view while using SwiftUI.

My desired functionality looked like this:

<img width="500" alt="Linnstrument Helper app notes pressed" src="https://user-images.githubusercontent.com/20845425/81235206-65a89680-8fc8-11ea-8263-c4213f40e66c.png"> 

As Anchor Preferences are a SwiftUI feature which is poorly documented by Apple, it was quite difficult figuring out the nuances of how it worked. A big shoutout goes to Javier of [SwiftUI Lab](https://swiftui-lab.com/) who wrote an [incredible tutorial series](https://swiftui-lab.com/communicating-with-the-view-tree-part-1/) for Anchor Preferences. For a detailed guide on the basics (and more advanced!) areas of Anchor Preferences, I highly recommend checking out Javier's series.

In addition to the series, Javier also answered a few questions I had via email, which is how I figured out how to display multiple instances of .overlayPreferenceValue() as well as use optionals to display no overlay when no MIDI notes are pressed.

## Multiple Anchor Preferences in SwiftUI

Although its fairly easy to implement a single anchor preference, things are a bit different when you want to display multiple items on a grid.

The first thing that I needed to do, was create a NoteBorder() view which contained the Anchor Preference code needed to build my border overlay. The code ended up looking like this:

```swift
struct NoteBorder: View {
    let color: Color
    let rect: CGRect
    
    var body: some View {
        RoundedRectangle(cornerRadius: 16)
               .strokeBorder(lineWidth: 4)
               .foregroundColor(color)
               .frame(
                   width: rect.width,
                   height: rect.height
           )
               .position(
                   x: rect.midX,
                   y: rect.midY
           )
    }
}
```

Next, I needed a way to map the MIDI notes to their respective locations on the grid, which I did inside my [MIDIEngine](https://github.com/markjamesm/linnstrument-helper/blob/master/LinnstrumentHelper/Model/MIDIEngine.swift) class in the mapSmallGridNotes(). Its rather long as I used if statements to handle each case. I imagine this could have been refined more by calculating note offsets on the grid, but this method worked well for now.

The last thing I needed to do was to create a .overlayPreferenceValue() inside my smallSurfaceView(). Using Foreach, I was able to create NoteBorder() instances for each element passed from my mapSmallGridNotes() method (notice how I use a different color for each distinct MIDI note):

```swift
.overlayPreferenceValue(GridItemBoundsPreferencesKey.self) { preferences in
     ZStack {
         ForEach(self.midiEngine.mapSmallGridNotes(note: self.conductor.note1), id: \.self) { note in
             NoteBorder(color: .blue, rect: preferences[note])
         }
         ForEach(self.midiEngine.mapSmallGridNotes(note: self.conductor.note2), id: \.self) { note in
            NoteBorder(color: .green, rect: preferences[note])
         }
         ForEach(self.midiEngine.mapSmallGridNotes(note: self.conductor.note3), id: \.self) { note in
            NoteBorder(color: .orange, rect: preferences[note])
         }
         ForEach(self.midiEngine.mapSmallGridNotes(note: self.conductor.note4), id: \.self) { note in
            NoteBorder(color: .red, rect: preferences[note])
         }
     }
}
``` 
Initially, I had tried putting my NoteBorder() code directly inside my smallGridView() struct, but that ended up causing XCode to timeout. According to Javier's answer to my email, 

>The ForEach compiler time-out was recognized by Apple in one of the release notes (at some point during the beta phase). The solution they provided there, was using custom views. Apparently the problem never got solved, but they decided to remove the advice from the release notes anyway.

With this code in place, I tested the program and here was the result when I pressed several notes at once:

<img width="750" alt="Linnstrument Helper app finished version" src="https://user-images.githubusercontent.com/20845425/81113716-aaf89580-8eee-11ea-8732-0b1a486deceb.png"> 

Success!

## Using Optionals When no Notes are Pressed

As the grid ranges from 0-127, we need to handle cases when no notes are pressed, and the best way to do so is to use optionals. Inside my Conductor class' receivedMIDINoteOff() method, I added the following code for each note pressed inside of the method's DispatchQueue:

```swift
DispatchQueue.main.async {
    self.note1 = nil
```

Now, whenever a note isn't pressed it will return nil. Repeat this code for every additional note you'd like to support.

## Conclusion

As a whole, I had a lot of fun working on this project. Not only was I able to broaden my SwiftUI skillset, I was also able to create a tangible program of use to people in general. If you have any questions regarding this guide, don't hesitate to [drop me a line!](https://markjames.dev/contact)
