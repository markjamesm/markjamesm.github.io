---
layout: post
title: Why I'm Learning F# in 2021
tags: [blog, programming projects] 
---

For the past while, I've had the goal of learning a functional programming language in the back of my mind but never acted on it for various reasons. However, during a recent weekend afternoon of winter COVID-lockdown web browsing, I stumbled across the terrific <a href="https://fsharpforfunandprofit.com/" target="_blank">F# for fun and profit</a> which piqued my interest and convinced me to take the plunge and learn FP in 2021. I believe that learning different programming paradigms helps you become stronger in all languages, and with C# incorporating more and more features from F# (records among other things for example) learning functional programming would be a benefit in a few different ways.

In addition, as I've been spending quite a bit of time working with C# and .NET 5 recently, F# felt like a natural fit due to the interoperability with the rest of the .NET ecosystem. In addition, F# has support for OO and so you can use a hybrid model in situations where pure FP may not be possible.

## Initial Thoughts

As a small learning project, I created a simple program which asks the user for a username and password and prints a message if the user enters the correct combination (if they get the combination wrong the function is called recursively and asks them for their credentials again). Figuring out the implementation details proved to be a bit more challenging than if I were to code the solution in C# as you don't want to use statements such as if and while in functional programming which would normally be my go-tos in a case like this. After much trial and error, I found my solution to be quite elegant:

```fsharp
open System

type LoginModel = {Username: string; Password: string}

// By making Admins a Set.OfList we can use .Contains later.
let Admins = [{Username="MJ";Password="M"}; {Username="Joe";Password="C"}] |> Set.ofList

[<EntryPoint>]
let main args =

    // The LoginRoutine() is a recursive function we call until the user enters the correct credentials.
    let rec LoginRoutine() = 
        printfn "Welcome to VMS Version V!"
        printf "Enter your username to continue: "
        let usernameAttempt = Console.ReadLine()
        printf "Enter your password to continue: "
        let passwordAttempt = Console.ReadLine()
        let loginCredentials = {Username=usernameAttempt; Password=passwordAttempt}
        match Admins.Contains loginCredentials with
        | true -> printfn "You are logged in"
        | false -> LoginRoutine()
    
    let login = LoginRoutine()

    0 // Terminate program
```

In comparison to C#, we see several big differences in the code. Aside from the lack of brackets and semicolons, the program uses a match expression rather than an if statement to evaluate the user's credentials. Pattern matches are powerful expressions and from what I've seen can allow for complex branching. 

Also note the creation of the Admins list:

```fsharp
let Admins = [{Username="MJ";Password="M"}; {Username="Joe";Password="C"}] |> Set.ofList
```

During the binding, we never specifed the actual type! F#'s powerful type inference means that I can skip a lot of boilerplate that I would normally use in C# while still having all the benefits of a strongly typed language. 

In addition, it is important to note that F# executes code from the top down which I find makes following program flow much easier. Despite this, I did run into a few compiler errors until I got the hang of the execution flow and making sure to put functions in the proper order. 

## The Bad
So far, my biggest issue hasn't been with the language itself, but rather the lack of learning resources compared with other, more popular languages. <a href="https://www.manning.com/books/get-programming-with-f-sharp" target="_blank">Get Programming with F#</a> has been a terrific book for learning, but the paradigm shift that functional programming requires can be quite significant and I find myself constantly Googling certain issues and being able to count the number of (usually not very related) Stack Exchange topics on one hand. I did however find a great small but active <a href="https://discord.gg/vs4U6HQhAG" target="_blank">F# Discord community</a> which has been great at answering smaller one-off questions that I've had regarding the language.

## Next Steps

My next steps are to think about a project that I'd like to undertake in F# to help me continue my learning. I find I tend to learn best when I have a project and write a build log documenting the process, and I'm hoping that I'll be able to combine an F# project with my <a href="https://markjames.dev/2020-12-09-back-to-school/" target="_blank">cloud computing program</a> that I'll be attending at the end of January. I also plan to write a few more articles exploring the language a bit more in-depth in the coming months, so stay tuned!   