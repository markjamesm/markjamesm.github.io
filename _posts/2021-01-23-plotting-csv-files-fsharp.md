---
layout: guide
title: Plotting Graphs from CSV Files in F# Using XPlot
tags: [guides, f#]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

Data visualization is an important tool, and there's been many cases where I've found myself wanting to visualize data from a CSV file. As I've [been learning F#](/2021-01-04-why-learning-fsharp-2021/) in the New Year, I thought that plotting a chart would be a great exercise to help sharpen my F# skills. Fortunately, charts can be created fairly easily in F# by using the [XPlot](https://fslab.org/XPlot/) library in conjunction with Plotly.

For the purposes of this exercise, I'll be using the [Spotify Dataset](https://www.kaggle.com/yamaerenay/spotify-dataset-19212020-160k-tracks?select=data_by_year.csv) from Kaggle, specifically the data by year file. From this file, we'll be looking to plot the danceability of music over time. According to the Kaggle definition, danceability is "The relative measurement of the track being danceable (ranging from 0 to 1)", but I'm unsure what the exact criteria for danceability is (maybe tempo and loudness combined with some other variable?). The end result should look something like this:

<img src="img/posts/graphing/music-danceability-over-time.png" width="710" height="601" alt="MusicSharp build featuring volume buttons">

# Reading a CSV File

After creating a new F# console project in Visual Studio, the first step was to load the CSV file so that we can feed the data to XPlot. This can be done using the CSV type provider in the [F# Data library](http://fsprojects.github.io/FSharp.Data/).

Next, I created a new module called CSVRead inside of a new file called CSVRead.fs. Inside the module, I place the first few lines of code:

```fsharp
open FSharp.Data

[<Literal>]
let filePath = """C:\Path\To\data_by_year.csv"""

type  rawCSV = CsvProvider<filePath, HasHeaders = true>
```

In the above code, we bind our filepath to an identifier and then create a new type which contains the CSVProvider plus our filepath. We also include HasHeaders = true as our CSV file contains column headers. One thing to note is that in F# we use [<Literal>] the same way we would const in C# or other languages. I'm using a literal here due to a blessing and curse of type providers in F#. On one hand, they're amazing, because you get compile-time types for your data! But, that also means the data must be available at compile time. You can usually work around this by either:
[*]Including representative data inside your project's git repo, so you can build the provider based on sample data and then parse any conforming input data
[*]Using a string literal in source code to define sample data and use that for the provider (which is what I've done here).

Now that we have our type provider setup, the next step is to create a function which loads our CSV file by using the GetSample() function from our type provider:

```fsharp
let loadCSVFile () =
    try
        let cSVFile = rawCSV.GetSample()
        let cSVFile = Some(cSVFile)
        cSVFile
    with _ -> None
```

This function's signature is unit -> Some cSVFile. Note that we're using an option here since the CSV file may not load (for more information on using options for error handling, see [this post](/2021-01-14-handling-errors-fsharp-with-option-types/)). 

# Charting the Data with XPlot

Now that we have a way to load our CSV file in place, the next step is to use XPlot to chart our data with Plotly, a handy charting library with an HTML frontend. To do so, I created a new file called Graphing.fs which would contain our graphing module and associated functions. 

With our Graphing module created, the first step was to create a graphData function which would map our chosen rows to a scatter plot. My solution ended up looking like this:

```fsharp
let graphData (rawCSV: rawCSV) =
    Scatter(
       x = [for row in rawCSV.Rows -> row.Year],
       y = [for row in rawCSV.Rows -> row.Danceability],
        mode = "lines+markers"
    )
```

This function takes in our rawCSV type and returns a Scatter object, which is produced by iterating over the Year and Danceability rows of the CSV file. 

One important thing to remember, is that in F#, the order of your files matter. In my Graphing module, I had been getting the error: [i]FS0039: The namespace or module 'CSVRead' is not defined[/i] which confused me as I was referencing was my own module inside Graphing.fs. It turns out however, that I had forgotten to put CSVRead.fs above Graphing.fs inside Visual Studio, but once I did the error cleared up right away. Although I'm not used to having the order of source files matter, it makes quite a lot of sense in terms of readability as the same principles apply to functions within the files. My experience with this style so far has made it easier to understand programs as I'm not constantly jumping from file to file and looking in different areas while trying to mentally keep track of function side effects.

The next step is to create a function which takes our scatter plot as input and then sets up the actual plot in Plotly:

```fsharp
let createPlot (data: Scatter) =
    data
    |> Chart.Plot
    |> Chart.WithOptions
        (Options(title = "Music Danceability Over the Ages"))
    |> Chart.WithXTitle("Year")
    |> Chart.WithYTitle("Danceability")
    |> Chart.WithWidth 700
    |> Chart.WithHeight 500
    |> Chart.Show 
```
The above code is fairy straightforward, setting up a pipeline which creates a chart to plot and then fills it with the relevant information. I must say that I'm becoming a big fan of F# pipe-forward operator as I find this style of coding to look clean and elegant while being easy to reason about.

# Creating the Graph

With our CSV and graphing modules now in place, the final step was to chain our functions together in our Program.fs EntryPoint:

```fsharp
open CSVRead
open Graphing

[<EntryPoint>]
let main argv =
    loadCSVFile()
    |> Option.map graphData
    |> Option.map createPlot 
    |> ignore
    0
```

Here, we're loading our CSV file and then chaining functions together to graph the data and create the plot. Note the use of Option.map here as loadCSVFile() is returning an option. In addition, we're ignoring the return value since Plotly is being loaded and displayed in the browser and we don't need any return value.

# Closing Notes

As you can see, plotting a chart from a CSV file is fairly straightforward in F#. I find F#'s pipe operator and lack of visual clutter makes it very succinct and easy to understand. Although this example creates a connect scatterplot, Plotly has a wide variety of chart types that you can create, and I highly recommend taking a look and playing around to see what you can do!