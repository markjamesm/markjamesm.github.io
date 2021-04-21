---
layout: post
title: On Learning F# — The First Month
tags: [f#, blog, guides]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

A little over a month ago, I made the decision to [learn F# in 2021](https://markjames.dev/2021-01-04-why-learning-fsharp-2021/). Since then, I've written a number [of guides](https://markjames.dev/tags/#f#) outlining some of my experiences while programming in FSharp. Recently, I've received a few emails from readers curious about developing in F#, and so I've decided to write a monthly blog post looking back at some of the things I've learned each month.  

## Learning to Think Functionally

During my first few weeks of learning, my biggest challenge was breaking out of the OOP paradigms that I had grown so accustomed to. Prior to starting F#, I had been programming with object oriented languages and was comfortable structuring programs in classes and managing state between objects. Working with pure functions was confusing at first, and I had trouble thinking about how I would structure programs. One thing that I found helped was [this terrific video](https://www.youtube.com/watch?v=SvOInBxPL30) by [Ben Gobeil](https://www.bengobeil.com/#about) which answered a lot of questions that I had. I really like that the video is annotated, and Ben is great at explaining some key functional concepts. For example, I had been experimenting with using the mutable keyword, but in Ben's video he notes that beginners should avoid it and that most programs can be written without it.

## A Functional Example using Records and Match Expressions

Using what I've learned so far, I've written a simple program which compares the top speed of two cars, here's how it looks:

```fsharp
// Defining the car type.
type Car =
    {
        Make: string;
        Model: string;
        TopSpeed: float;
    }

// Creating two cars to race.
let carOne =  { Make = "Ferrari"; Model = "488 GTE"; TopSpeed = 490.0 }
let carTwo = { Make = "McLaren"; Model = "F1"; TopSpeed = 386.4 }


let race (carOne: Car) (carTwo: Car) =
    // Pattern match the potential race outcomes.
    let raceOutcome race =
        match race with
        | ("Ferrari") -> printfn "Ferrari wins!"
        | ("McLaren") -> printfn "The McLaren won!"
        | ("Tie!") -> printfn "It was a tie!"
        | (_) -> printfn "Invalid result."

    // The race logic.
    let racing =
        if (carOne.TopSpeed > carTwo.TopSpeed) then
            carOne.Make
        elif (carOne.TopSpeed = carTwo.TopSpeed) then
            carTwo.Make
        else
            "Tie!"

    raceOutcome racing

[<EntryPoint>]
let main argv =
    race carOne carTwo
    0 // return an integer exit code
```
In this example, there are several things to note. Firstly, we define a Record type which we can use to model each of the cars. The benefit to using record types is that we can compare values between records out of the box. This is great, as normally in C# you would need to conform to IEquatable to perform this type of comparison. 

Secondly, note that we're using pattern matching as opposed to using an if statement. We do use an if expression to compare the records and return a value, but we then match on the resulting value rather than using another if. In C# you can use if statements to return different values, but in F# if statements must always evaluate to the same type. Initially, I found myself struggling with this concept, but as I continued to write small practice scripts I realized that working with expressions makes your code much easier to reason about.

The final thing to note is the order of the code. In F#, the order of code and files matters, and the above code is organized based on dependencies. For example, the racing function depends on the raceOutcome modules to analyze the results, and so the raceOutcome function needs to be placed above. I find that structuring programs this way makes the code easier to reason about (especially when looking at the code of others) since you can start at the bottom of a file and work your way up to see how everything interacts.

## Going Forward

At work, I've been increasingly working on analytics and performance benchmarking, and I see F# as being an excellent tool in these domains. So far, Type Providers have made loading CSV files a breeze, and I'm currently looking into using [Deedle](https://bluemountaincapital.github.io/Deedle/index.html) to create dataframes for further exploratory analysis. To this end, I've been using .NET Interactive Notebooks and have also experimented with [iRacing Telemetry in F#](/2021-02-09-iracing-telemetry-fsharp/) which has helped me get used to using them.

## Some Tips

* Learn to appreciate higher order functions, and take the time to learn some of the most useful ones (Map, Seq.Filter, Seq.Iter) as they can be incredibly powerful while developing. I recommend testing out Map, Seq.filter, Seq.iter in the REPL to familiarize yourself with what they do and how they can work together.

* The learning curve can be steep at times (especially if you're coming from an OOP background), and sometimes taking a break can help you approach a problem differently. For example, I had been stuck thinking about a problem using a for loop for several hours a couple weeks ago, and so I went for an hour long walk. When I got in, I sat down and suddenly the concept of Seq.map and Seq.filter became much more clear.