---
layout: guide
title: On Transitoning from C# to F# — Rethinking State
tags: [f#, guides]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

In programming (or life for that matter!) shifting paradigms can be hard, requiring you to adopt a whole new mindset and way of doing things. Since I've only ever really worked with object oriented programming languages, starting to [learn F#](/2021-01-04-why-learning-fsharp-2021/) and its functional programming ways has been quite a shift. 

As a result of my ongoing experiences, I've decided to create a series of articles in which I compare and contrast different elements of funtional and object oriented (OOP) programming paradigms and explore the best way to handle certain cases from a functional standpoint. Although the code examples will be in C#, the series is written in a way as to be of reference for anyone coming from an OOP language and looking to experiment with functional programming. 

# Rethinking State

For me, one of the biggest changes in learning to think in a functional manner was to rethink how I approach state in an application. Prior to starting to learn F#, I had always been confused when I heard the terms "pure function" or "side effect" get thrown around. I knew that functional programming languages made use of pure functions and that side effects were undesirable, but I had no idea why. A good way to explain these concepts is to write a bit of code which explores state in a simple way. 

For example, say you're creating a simple action game and want to have a player fight an enemy who deals damage. In C#, you would most likely create a player health property and modify it with a method like so:

```csharp
public class Player
{
    public int PlayerHealth { get; set; } = 100;
    
    public void FightEnemy()
    {
        this.PlayerHealth -= 5;
    }
}
```

We can then call the method where we need to elsewhere in the code:

```csharp
Player player = new Player();
// PlayerHealth is 100.
player.FightEnemy();
// PlayerHealth is now 95.
Console.WriteLine(player.PlayerHealth);
```
By creaintg a state in the PlayerHealth property and then modifying it using the void FightEnemy() method, we're producing what's known as a [i]side effect[/i]. According to Wikipedia, the formal definition for a side effect is:

```
... An operation, function or expression is said to have a side effect if it modifies some state variable value(s) outside its local environment, that is to say has an observable effect besides returning a value (the main effect) to the invoker of the operation.
```

This essentially means that you want to avoid using functions that modify values outside of them, but rather return a value instead. 

The issue with having side effects is that it makes your code much harder to reason about. Now, this isn't always a problem in smaller examples of code like this, but imagine a large and mature codebase with thousands of functions. How would you know what a method does if it doesn't produce a result? What if multiple methods act on the same property?

Now, let's take a look at how we could reproduce this code in a functional way in F# and avoid side effects:

```fsharp
let fightEnemy playerHealth =
    playerHealth - 5   
```
Here, we're creating what's known as a pure function which takes the input playerHealth and then returns the playerHealth - 5. A pure function is only determined by the values of its input and does not cause any side effects (as the above C# code did). Given the same input, no matter how many times we call our fightEnemy function the result will always be the same.

Next, we can use the fightEnemy method by binding playerHealth to a value and then passing it to fightEnemy:

```fsharp
let playerHealth = 100
// Returns 95.
fightEnemy playerHealth 
```

In contrast to the void C# method which modifies a playerHealth property, in F# we pass fightEnemy our playerHealth value and it gives us a playerHealth value in return, producing no side effects in the process. The function depends on the input in order to produce a result which we can then use elsewhere in our code. Aside from making the function much easier to reason about, it also has the added benefit of making your code much easier to debug and test as functions exist independent of one another and the outside world. 

# Summary Notes

As you can see, there are a number of benefits to using pure functions and avoiding mutable state in your programs. Even just looking at the F# code, I find things much easier to reason about since I don't need to keep track of a class property which may or may not be changed by multiple side effect producing methods. Of course, there are some instances where side effects can't be avoided (eg: writing to a database or gettting an HTTP file), and this makes it important to design your functional code in such a way as to push the side effect producing code to the boundaries of your application. 