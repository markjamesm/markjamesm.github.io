---
layout: guide
title: Writing an iRacing SDK Implementation in F# (Part One)
tags: [guides, programming projects, f#]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

In my [previous post](/2021-01-04-why-learning-fsharp-2021/), I discussed how I've decided to learn F# in 2021 for a number of reasons. Around the same time, I also happened to setup my Sim Racing rig so that I could continue to play <a href="https://www.iracing.com/" target=_blank>iRacing</a> with my VR headset (HTC Vive). Its been several years since I've last played, but with COVID-related curfews being implemented here in Montreal tomorrow, I've been increasingly taking up home-based pursuits which I didn't always have the time for pre-lockdown. Since the last time I played iRacing, I'm running a PC with a much better processor, motherboard, and only SSDs. The VR performance has been a huge leap forward since I used to play with my old machine and I was quite impressed. After spending a couple of hours setting up, here's what my current humble racing setup:

<img src="/img/posts/irsdk-fsharp/racing-rig.jpg" width="500" height="375" alt="Photo of my current iRacing VR setup">

Having a dedicated table really helps, as in my old apartment it was fairly difficult to setup a station with limited space, but now I can fortunately just jump in. Despite the past limitations, I was still able to get fairly competitive and still remember the thrill of my first win agaisnt a field of real racers:

<img src="/img/posts/irsdk-fsharp/iracing-win.jpg" width="500" height="279" alt="Photo of my iRacing first win certificate">

Inspired by setting everything up and doing some laps to practice for an eventual return to comeptition, I started thinking about how I once experimented with using the <a href="https://github.com/kutu/pyirsdk" target=_blank>Python implementation</a> of the iRacing SDK to connect to an arduino and display a speedometer readout in realtime on a small screen. In reminiscing about the experience, I thought about how I could look into writing an F# implementation of the SDK as a learning project. In addition to learning through the project, it also has the benefit of being of use in a future project involving an iRacing stats tracker web app that I've been thinking about writing as a project for my upcoming [cloud computing courses](/2020-12-09-back-to-school/). 

## Getting Started

I tend to learn best when projects are slightly outside of my comfort zone, and this would be both my first time writing a library, as well as writing one in a functional language! Having used an array of libraries at this point, I had some confidence in choosing an organizational structure, and the Python implementation is only <a href="https://github.com/kutu/pyirsdk/blob/master/irsdk.py" target=_blank>739 lines of code</a> which felt doable compared to some of the larger libraries out there.

Moreover, the python implementation of the SDK has the ability to: 

* Get session data (WeekendInfo, SessionInfo, etc...)
* Get live telemetry data (Speed, FuelLevel, etc...)
* Broadcast messages (camera, replay, chat, pit and telemetry commands)

and I figured that this would be a good featureset to aim for in the final version of the F# SDK. Out of these features, the session data and live telemetry data would be the ones I plan to implement first.

## Creating the Library

After coming up with some desired features, the first step was to create a new FSharp solution called iRacingFSharp. Inside the solution, I created two projects. One was our actual library, called iRacingFSharp, and the other was a basic console app called SDKReader (located in the Examples Folder) to test the functionality of the library as I worked on it.

## The First Function

Starting small, I decided that a good first function would be to find out the state of the simulator. Fortunately, the iRacing SDK allows you to check if the sim is running using the following URL which points to a localhost server:

```html
http://127.0.0.1:32034/get_sim_status?object=simStatus
```
Getting this URL in Postman returns a JSON object which looks like this:

```json
var simStatus={
   running:0 // 1 if the sim is running
};
```

I decided to make use of the <a href="https://fsharp.github.io/FSharp.Data/library/Http.html" target=_blank>F# Data HTTP library</a> in order to download the response and so I installed it from NuGet at this point. 

Next, inside my iRacingFSharp project I created a file called Irsdk.fs and wrote the following code: 

```fsharp
namespace IrsdkFS

open FSharp.Data

///<summary>F# implementation of the iRacing SDK.</summary>
module IrsdkFS =

    ///<summary>Returns the simStatus in string format</summary>
    let SimStatus() =
        let simStatusURL = "http://127.0.0.1:32034/get_sim_status?object=simStatus"
        let simStatusObject = Http.RequestString(simStatusURL)
        simStatusObject
```

In the above code, I've created a module which contains a function called SimStatus that takes no parameters. It then binds the JSON response to simStatusURL and passes it to the HTTP library via Http.RequestString(). Finally, simStatusObject is returned in string format which can be parsed further by another function in a later step.

With this simple function in place, the next step was to create SDKReader.fs inside my SDKReader console app. This file contained code to call the SimStatus() function and print the output:

```fsharp
[<EntryPoint>]
let main argv =
    let test = IrsdkFS.SimStatus()
    printf "%s" test
    0 // return an integer exit code
```

Running dotnet build inside the SDKReader folder displayed the following output while iRacing was running:

```json
"var simStatus={
   running:1
};
"
```

Success! With this method working, we now have the very beginnings of an F# implementation of the iRacing SDK! Although it is a small step, we were also able to create and structure the project. Lastly, I also setup a a basic .NET build through Github Actions for CI. 

Stay tuned for Part Two where I plan to implement some telemetry functions, and be sure to follow along with the <a href="https://github.com/markjamesm/irsdk-fsharp" target=_blank>Github Repo here</a> if you're interested in seeing how to project progresses (or would like to contribute)!