---
layout: guide
title: Error Handling in F# with Option Types — A Practical Example
tags: [f#, guides]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

As I continue to study functional programming with F#, I've been learning to handle things (such as [thinking about state](/2021-01-11-transitioning-from-csharp-to-fsharp-rethinking-state/)) in a different way than I would in an object oriented programming language. In keeping with my recent learning F# article series, I've decided to discuss another element of the learning process which had caused me some confusion over the past few days - namely error handling in F#. 

Although the example I will be using deals with opening a memory mapped file, it is applicable to any kind of setting where you might return either none or a value.

## IO Handling in F#

For a learning project, I've been working on implementing the [iRacing SDK in F#](/2021-01-08-writing-an-iracing-sdk-implementation-fsharp/). Early on in development, one of the first things I needed to do was some IO, namely loading a [MemoryMappedFile](https://docs.microsoft.com/en-us/dotnet/standard/io/memory-mapped-files) so that I could get realtime data from the sim. Technically, this can be done in a single line of code:

```fsharp
let memoryMap = MemoryMappedFile.OpenExisting("Local\\IRSDKMemMapFileName") 
```

However, opening a MemoryMappedFile requires the iRacing simulator to be running. If MemoryMappedFile.OpenExisting() does not find the desired MemoryMap, then it fails with a System.IO.FileNotFound exception. No problem! in C# I would simply wrap the error in a try/catch block and fail gracefully if no file is found, and so I figured that I could do the same here in F#:

```fsharp
let validateMemoryMappedFile () = // This function returns an option of type 'a. (string -> 'a option)
    try
        let memoryMap = MemoryMappedFile.OpenExisting("Local\\IRSDKMemMapFileName") 
        Some(memoryMap)
    with
        | ex -> eprintf "Error: %s" ex.Message 
                None
```

In the above code, we try to bind memoryMap with the MemoryMappedFile, and if the operation is successful we then return Some memoryMap. In F#, the Some keyword is used to implement what's known as an *Option*. The Option type is F#'s alternative to null, and behind the scenes its a discriminated union that looks like this:

```fsharp
type Option<'a> = //'a represents a generic type  
   | Some of 'a   // A value of type 'a exists        
   | None         // No value exists
```

Now when I tested my code, everything seemed to work as planned with validateMemoryMappedFile either returning Some MemoryMappedFile (-> 'a option) or failing with an error message and then returning None. Success, right?! No so fast! The next step in my chain was to create a ViewAccessor from the MemoryMap so that I could get ready to read the data. The pure function to do this looks like:

```fsharp
let private loadMemoryMap (memoryMappedFile: MemoryMappedFile) =
    let iRacingMemoryMapAccessor = memoryMappedFile.CreateViewAccessor()
    iRacingMemoryMapAccessor
```

Notice that loadMemoryMap takes a MemoryMappedFile and not an option type. If we wanted, we could implement match checks in our function to make sure we have a MemoryMappedFile, but that would include a lot of boilerplate inside our functions as we pass the MemoryMap down the chain. Fortunately, we're going to explore a much cleaner alternative which doesn't involve as boilerplate.

## Introducing Railway Oriented Programming

On the terrific F# for Fun and Profit website, there's an [in-depth article](https://fsharpforfunandprofit.com/posts/recipe-part2/) explaining the concept of Railway Oriented Programming which I highly recommend reading. This guide helped to explain the concept of Railway Oriented Programming, which is to create two "tracks" that functions have, one for the expected result, and one to catch any errors. When an error occurs, it gets shunted off the good chain and into the error chain. Now, I understood the concept of the idea (the railway analogy is super helpful), but I was having trouble implementing it in practice.

The main sticking points for me were the idea of the error track and not catching an exception in the objected oriented sense. In addition, I was getting confused since you can use Options or Results to handle errors in Railway Oriented Programming and I wasn't exactly sure which to use at the time.

Fortunately, a super helpful user (nffa) on the F# Slack channel was able to help me in understanding the concept by working through some example code with me and sharing some helpful tips. Although the F# community is relatively small, I will say that the answers to questions I've asked in the Slack and Discord channels have been absolutely terrific! 

As it turns out, in order to properly implement Railway Oriented Programming, I needed to make use of Option types in conjunction with map and bind to really get the magic of Railway Oriented Programming. As we will see, map and bind cut down significantly on boilerplate and enable us to call loadMemoryMap without needing to implement any error checking inside of it. The first step was to create a new function called start () which would setup the pipeline:

```fsharp
///<summary>Loads the iRacing memory map file if it is present on disk.</summary>
let start () =
    openMemoryMappedFile
    |> Option.map loadMemoryMap // If openMemoryMappedFile is some, then call loadMemoryMap
```

Inside our start function, we call openMemoryMappedFile which either returns Some MemoryMappedFile or None. Next, we use the pipe forward operator to pass the MemoryMappedFile to loadMemoryMap, but only if we have Some MemoryMappedfile.

The Option.map flow looks like this:

```fsharp
> Option.map;;
val it : (('a -> 'b) -> 'a option -> 'b option)
```

But what happens if the loadMemoryMap function needs to return an error? Much like how we used Option.map to pass our optional MemoryMap to our loadMemoryMap function, we can use Option.bind to bind the option to the existing option in our chain! Here's what the Option.bind looks like:

```fsharp
> Option.bind;;
val it : (('a -> 'b option) -> 'a option -> 'b option)
```

Notice how similar it looks to Option.map, with the exception being the second item is of type 'b option rather than 'b. Bind is like map, but where the mapping function returns the same type of context that the source value is.

## Additional Notes Regarding Option Types

Conceptually, Option and Result are mostly the same, except that Result carries information about the nature of why something failed. If this isn't important, then Option works as well. Moreover, in the [Against Railway Oriented Programming](https://fsharpforfunandprofit.com/posts/against-railway-oriented-programming/) section, Scott recommends against using Results for IO in F#, as

*anywhere that there is I/O there will many, many things that can go wrong. It is tempting to try to model all possibilities with a Result, but I strongly advise against this. Instead, only model the bare minimum that you need for your domain, and let all the other errors become exceptions.*

From what I understand, those new to functional languages should focus on using the Option type and only start using results when explicitly needed. As options work in most cases, you'll likely be quite comfortable in the language by the time you start making use of Result types and by then their use should be trivial, especially since Options can become Results as long as you can 'fill in' the missing Error type/data.  Moreover, Results can become options by just dropping their error data.

## Conclusion

Hopefully this guide helped clear some things up with regards to Railway Oriented Programming and error handling in F#. I had been wrapping my head around this concept for days before it finally clicked for me, and I hope that I can pass some of the learning experience on to others who may be running into similar issues. I'm glad I stuck with it though, as compared to languages such as C# or Java, it is clear to see how much error boilerplate code we can omit as we no longer need to check for null at every step in the program flow!