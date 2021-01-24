---
layout: guide
title: Plotting Graphs from CSV Files in F# Using XPlot
tags: [guides, f#]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

Data visualization is an important tool, and there's been many cases where I've found myself wanting to visualize data from a CSV file. As I've [been learning F#](/2021-01-04-why-learning-fsharp-2021/) in the New Year, I thought that plotting a chart would be a great exercise to help sharpen my F# skills. This is especially true as once I finish porting[the iRacing SDK]](/2021-01-08-writing-an-iracing-sdk-implementation-fsharp/) to F#, the next step will be to record my lap times and visualize the telemetry data. Fortunately, charts can be created fairly easily in F# by using the [XPlot](https://fslab.org/XPlot/) library in conjunction with Plotly.

For the purposes of this exercise, I'll be using the [Spotify Dataset](https://www.kaggle.com/yamaerenay/spotify-dataset-19212020-160k-tracks?select=data_by_year.csv) from Kaggle, specifically the data by year file. From this file, we'll be looking to plot the danceability of music over time. According to the Kaggle definition, danceability is "The relative measurement of the track being danceable (ranging from 0 to 1)", but I'm unsure what the exact criteria for danceability is (maybe tempo and loudness combined with some other variable?). The end result should look something like this:

<img src="/img/posts/graphing/Music-danceability-over-time.png" width="710" height="601" alt="Graph showing music's danceability from 1920-2020">

<html>
    <head>
        <meta charset="UTF-8" />
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    </head>
    <body>
        <div id="31c5f812-0967-4fb6-ad2c-9e1a6ca712bc" style="width: 700px; height: 500px;"></div>
        <script>
            var data = [{"type":"scatter","x":[1921,1922,1923,1924,1925,1926,1927,1928,1929,1930,1931,1932,1933,1934,1935,1936,1937,1938,1939,1940,1941,1942,1943,1944,1945,1946,1947,1948,1949,1950,1951,1952,1953,1954,1955,1956,1957,1958,1959,1960,1961,1962,1963,1964,1965,1966,1967,1968,1969,1970,1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017,2018,2019,2020],"y":[0.4185973333333336,0.4820422535211267,0.5773405405405401,0.5498940677966102,0.5738633093525181,0.5998802612481859,0.6482682926829262,0.5342878667724027,0.6476698529411761,0.5181758835758836,0.5952217391304357,0.5577976095617526,0.57029030390738,0.528705882352941,0.5558691699604746,0.5580055454545453,0.5421572298325723,0.47997797716150103,0.5126828,0.52189235,0.4804813541666676,0.4646338882282997,0.4551456338028168,0.5001744680851068,0.5191431500000011,0.4144450116009276,0.4713208484848491,0.4633694736842109,0.4421993999999996,0.5042531000000008,0.4624800999999999,0.4570322000000001,0.4374260512820513,0.4656388999999994,0.4881312,0.4878145000000007,0.5034812307692305,0.4800213999999995,0.4821143076923083,0.4860292432432435,0.4891655789473678,0.4931643684210523,0.48162135,0.5041769743589742,0.5034505641025638,0.5070204999999981,0.4926885942971483,0.5010080999999997,0.4880977999999999,0.5063075000000004,0.5042738499999999,0.5209994499999997,0.5154494500000005,0.51533975,0.5209980000000007,0.5297624999999999,0.5333237000000017,0.5407746000000005,0.5620453000000001,0.5561523589743594,0.5421846999999999,0.5642594000000005,0.5456292999999998,0.5303089499999987,0.5556424000000012,0.5409334000000001,0.5410193333333342,0.5404594358974372,0.5472274499999988,0.5352986500000001,0.5558243589743591,0.5550648499999989,0.5698781500000001,0.5528299500000016,0.5590457500000001,0.5878306000000001,0.5768138499999994,0.5862116499999998,0.5821579190158892,0.590918047034764,0.5833178553615969,0.5761602999999997,0.5757633060388944,0.5676803662258397,0.5722805641025652,0.5682301538461539,0.5634143589743592,0.5791928388746803,0.5641903571428577,0.5724883432539687,0.5528669806643526,0.5708818508997433,0.5711480263157896,0.5899476807980057,0.5937740628166152,0.6002023928770179,0.6122170180722886,0.6635004755111744,0.6448141097998967,0.6929043349753701],"mode":"lines+markers"}];
            var layout = {"title":"Music Danceability Over the Ages","xaxis":{"title":"Year","_isSubplotObj":true},"yaxis":{"title":"Danceability","_isSubplotObj":true}};
            Plotly.newPlot('31c5f812-0967-4fb6-ad2c-9e1a6ca712bc', data, layout);
        </script>
    </body>
</html>

# Reading a CSV File

After creating a new F# console project in Visual Studio, the first step was to load the CSV file so that we can feed the data to XPlot. This can be done using the CSV type provider in the [F# Data library](http://fsprojects.github.io/FSharp.Data/).

Next, I created a new module called CSVRead inside of a new file called CSVRead.fs. Inside the module, I place the first few lines of code:

```fsharp
open FSharp.Data

[<Literal>]
let filePath = """C:\Path\To\data_by_year.csv"""

type  rawCSV = CsvProvider<filePath, HasHeaders = true>
```

In the above code, we bind our filepath to an identifier and then create a new type which contains the CSVProvider plus our filepath. We also include HasHeaders = true as our CSV file contains column headers. 

One thing to note is that in F# we use \[\<Literal\>\] the same way we would const in C# or other languages. I'm using a literal here due to a blessing and curse of type providers in F#. On one hand, they're amazing, because you get compile-time types for your data! But, that also means the data must be available at compile time. You can usually work around this by either:
* Including representative data inside your project's git repo, so you can build the provider based on sample data and then parse any conforming input data
* Using a string literal in source code to define sample data and use that for the provider (which is what I've done here).

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

One important thing to remember, is that in F#, the order of your files matter. In my Graphing module, I had been getting the error: *FS0039: The namespace or module 'CSVRead' is not defined* which confused me as I was referencing was my own module inside Graphing.fs. It turns out however, that I had forgotten to put CSVRead.fs above Graphing.fs inside Visual Studio, but once I did the error cleared up right away. 

Although I'm not used to having the order of source files matter, it makes quite a lot of sense in terms of readability as the same principles apply to functions within the files. My experience with this style so far has made it easier to understand programs as I'm not constantly jumping from file to file and looking in different areas while trying to mentally keep track of function side effects.

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