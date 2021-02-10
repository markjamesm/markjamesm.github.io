---
layout: guide
title: iRacing Telemetry With F#
tags: [f#, guides, iracing]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

During the pandemic, I've been spending quite a bit of time playing [iRacing](https://iracing.com) in VR. I love the realism of iRacing (and how well it supports VR!), and managed to score a win in the Mazda MX-5 this season at the (sadly) now defunct Oran Park Raceway near Sydney. 

<img src="/img/posts/race-result-mazda.png" width="600" height="150" alt="First place at Oran Park Raceway">

In a [previous post](https://markjames.dev/2021-01-08-writing-an-iracing-sdk-implementation-fsharp/), I discussed how I was looking to create an F# library for the iRacing SDK. This proved to be a bigger challenge than I thought as the iRacing API uses a memory mapped file and I'm not yet familiar enough with F# to be tackling the problem yet.

As an alternative (and thanks to F#'s interoperability with C# and the .NET platform), I decided to familiarize myself with this [C# iRacing SDK](https://github.com/NickThissen/iRacingSdkWrapper) and build a small app that gathers some basic telemetry data and writes it to a CSV file which I could then plot in a .NET Interactive Notebook using Plotly.NET. The end result of my first few tests looked like this:

<html>
    <head>
        <!-- Plotly.js -->
        <meta http-equiv="X-UA-Compatible" content="IE=11" >
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
        <style>
        .container {
          padding-right: 25px;
          padding-left: 25px;
          margin-right: 0 auto;
          margin-left: 0 auto;
        }
        @media (min-width: 768px) {
          .container {
            width: 750px;
          }
        }
        @media (min-width: 992px) {
          .container {
            width: 970px;
          }
        }
        @media (min-width: 1200px) {
          .container {
            width: 1170px;
          }
        }
        </style>
    </head>
    <body>
      <div id="c47f4f06-1442-40f6-8bba-417275319a5f" style="width: 750px; height: 400px;"><!-- Plotly chart will be drawn inside this DIV --></div>
<script type="text/javascript">

            var renderPlotly_c47f4f06144240f68bba417275319a5f = function() {
            var fsharpPlotlyRequire = requirejs.config({context:'fsharp-plotly',paths:{plotly:'https://cdn.plot.ly/plotly-latest.min'}}) || require;
            fsharpPlotlyRequire(['plotly'], function(Plotly) {

            var data = [{"type":"scatter","x":[0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.17,0.18,0.18,0.18,0.19,0.19,0.2,0.2,0.21,0.21,0.22,0.22,0.23,0.23,0.24,0.24,0.25,0.25,0.26,0.27,0.27,0.28,0.28,0.29,0.3,0.3,0.31,0.31,0.32,0.33,0.33,0.34,0.35,0.35,0.36,0.37,0.38,0.39,0.39,0.4,0.41,0.41,0.42,0.42,0.43,0.43,0.43,0.44,0.45,0.45,0.46,0.46,0.47,0.48,0.48,0.49,0.5,0.5,0.51,0.52,0.53,0.53,0.54,0.54,0.55,0.56,0.56,0.57,0.57,0.58,0.59,0.6,0.6,0.61,0.62,0.63,0.63,0.64,0.65,0.66,0.66,0.67,0.67,0.68,0.68,0.69,0.69,0.69,0.69,0.7,0.7,0.7,0.71,0.71,0.72,0.72,0.73,0.74,0.74,0.75,0.76,0.76,0.77,0.77,0.78,0.79,0.79,0.8,0.81,0.81,0.82,0.82,0.83,0.83,0.84,0.85,0.85,0.86,0.86,0.87,0.88,0.89,0.89,0.9,0.9,0.91,0.91,0.91,0.91,0.92,0.92,0.93,0.93,0.94,0.94,0.95,0.95,0.96,0.96,0.97,0.98,0.99,0.99],"y":[0,0,0,0,0,0,0,0,0,0,9,20,28,37,48,59,72,85,98,111,121,127,124,121,118,114,110,113,123,132,141,150,156,162,148,140,136,133,135,142,148,148,154,161,168,174,180,185,190,194,198,202,202,183,164,144,125,113,106,108,114,121,129,137,145,152,157,163,169,174,179,183,187,180,164,150,147,143,144,151,157,163,169,173,177,181,183,185,189,192,195,198,198,178,155,143,135,118,99,82,70,66,72,82,91,97,103,113,124,135,145,153,152,150,148,145,141,140,147,154,162,169,175,164,148,140,138,136,138,142,146,152,159,166,173,179,177,153,127,99,77,67,68,70,80,92,107,118,127,137,146,153,160,167,174,180,185,189],"mode":"lines","line":{"width":{},"shape":"spline"},"marker":{}}];
            var layout = {"title":"Laguna Seca (Ferrari 488 GTE)","xaxis":{"title":"Lap Distance (%)","showgrid":false,"position":200.0},"yaxis":{"title":"Speed (Km/h)","showgrid":false},"width":750.0,"height":400.0};
            var config = {};
            Plotly.newPlot('c47f4f06-1442-40f6-8bba-417275319a5f', data, layout, config);
});
            };
            if ((typeof(requirejs) !==  typeof(Function)) || (typeof(requirejs.config) !== typeof(Function))) {
                var script = document.createElement("script");
                script.setAttribute("src", "https://cdnjs.cloudflare.com/ajax/libs/require.js/2.3.6/require.min.js");
                script.onload = function(){
                    renderPlotly_c47f4f06144240f68bba417275319a5f();
                };
                document.getElementsByTagName("head")[0].appendChild(script);
            }
            else {
                renderPlotly_c47f4f06144240f68bba417275319a5f();
            }
</script>     
    </body>
</html>

The entire process was refreshingly simple, but there were a few things that had stumped me as I had still been thinking in C# (which I'm more familiar with). For example, here's the code which listens to an event from the iRacing SDK and then calls a function which parses and then writes telemetry data to a CSV: 

```fsharp
// Get the current speed in m/s and convert to a rounded km/h.
let getSpeed (evArgs: SdkWrapper.TelemetryUpdatedEventArgs) =
    let speed = float evArgs.TelemetryInfo.Speed.Value
    let speedInKMh = speed * 3.6
    let speedRounded = System.Math.Round (speedInKMh, 0) 
    speedRounded

// Get the throttle input and round it.
let throttleValue (evArgs: SdkWrapper.TelemetryUpdatedEventArgs) =
    let throttle = float evArgs.TelemetryInfo.Throttle.Value
    let roundedThrottle = System.Math.Round (throttle, 2)
    roundedThrottle

// Get the lapdistance and round it.
let getLapDistance (evArgs: SdkWrapper.TelemetryUpdatedEventArgs) =
    let distance = float evArgs.TelemetryInfo.LapDistPct.Value
    let roundedDistance = System.Math.Round (distance, 2)
    roundedDistance

// Append the output of the current tick to a CSV file.
let writeToCsv (evArgs: SdkWrapper.TelemetryUpdatedEventArgs) =
    let dataToWrite = $"{getSpeed evArgs},{evArgs.TelemetryInfo.Gear.Value},{throttleValue evArgs},{evArgs.TelemetryInfo.Lap.Value},{getLapDistance evArgs}\n"
    File.AppendAllText ("""C:\LapTimes.csv""", dataToWrite)
    printfn "Wrote to CSV"

let start () =
    // Bind an instance of iRacing SdkWrapper to iRacing
    let iracing = new SdkWrapper(TelemetryUpdateFrequency = 2.0)
    iracing.TelemetryUpdated
    // Listen to the iRacing telemetry updates
    |> Observable.subscribe writeToCsv |> ignore
    iracing.Start()
```
The first thing to note is the structure of the code. F# code is structured from the bottom up (the same applies for files in a project), and I find that this helps with readability and reasoning about the code.

Another thing to consider is how to set the update frequency (in updates per second). In C#, we would update a variable to set the telemetry update frequency like so:
```csharp
iracing.TelemetryUpdateFrequency = 2.0; // Updates per second
```
However, In F#, we bind names to expressions as opposed to assigning variables. Then how can we set an update frequency for the C# library without using a mutable? The answer is simple, pass it in as a parameter to the SdkWrapper constructor:

```fsharp
let iracing = new SdkWrapper(TelemetryUpdateFrequency = 2.0) // Receive events at a rate of two updates/sec.
```

Next, note the use of Observable.subscribe to listen to an event. In C#, we would likely do something like this to subscribe to the event using an event handler:

```csharp
class Program 
{
    private readonly Sdkwrapper iracing;

    static void Main(string[] args)
    {    
        iracing = new SdkWrapper;
        iracing.TelemetryUpdated += OnTelemetryUpdated;
        iracing.Start();
    }

    private void OnTelemetryUpdated(object sender, SdkWrapper.TelemetryUpdatedEventArgs e)
    {
        // Use live telemetry
    }
}
```

In F#, we subscribe to the event using Observable.subscribe and call our writeToCsv function each time the event is fired:
```fsharp
    iracing.TelemetryUpdated
    |> Observable.subscribe writeToCsv |> ignore
```

The interop between C# and F# events is nice, and I find this solution to be quite elegant. In addition, you also have Observable.scan which accumulates state each time an event has fired. I haven't had to do that in this example, but you can see [more details here](https://fsharpforfunandprofit.com/posts/concurrency-reactive/).

## Plotting the Telemetry Data

Now that we have a backend system logging some telemetry data from the sim, the next step is to plot the graph we saw above. In order to do so, I created a new .NET Interactive notebook in VS Code Insiders. I then imported the libraries I needed and wrote the following initial code:
```fsharp
[<Literal>]
let FilePath = """C:\LapTimes.csv"""

type rawCsv = CsvProvider<FilePath, HasHeaders = true>

// CSV File
let lapPerformance = rawCsv.GetSample()
```
The use of \[\<Literal\>\] here is because FilePath must be a constant so that the CsvProvider can read the data while we're developing. As I mentioned in a previous article on [CSV files in F#](/2021-01-23-plotting-csv-files-fsharp/), "Type providers are a blessing and curse in F#. On one hand, they're amazing, because you get compile-time types for your data! But, that also means the data must be available at compile time. You can usually work around this by either:
* Including representative data inside your project's git repo, so you can build the provider based on sample data and then parse any conforming input data
* Using a string literal in source code to define sample data and use that for the provider (which is what I've done here)."

 I also added some column headers into my CSV file (in the future, I plan to write these automatically in the backend code). We also create a new CsvProvider, and then get the data by calling GetSample() on the raw CSV file.

Next, let's write the createChart function and then pass it out formatted CSV file:

```fsharp
let createChart(lapPerformance: rawCsv) = 

    let speed = lapPerformance.Rows |> Seq.filter (fun row -> row.Lap = 2 )
    speed
    // Map Lap Distance and Speed to X and Y on the line chart.
    |> Seq.map (fun row -> row.LapDistance, row.Speed) 
    |> Chart.Spline
    |> Chart.withTitle "Laguna Seca (Ferrari 488 GTE)"
    |> Chart.withX_AxisStyle ("Lap Distance (%)", Showgrid=false,Position=200.0)
    |> Chart.withY_AxisStyle ("Speed (Km/h)", Showgrid=false)
    |> Chart.withMargin(Margin.init(120, 100, 50, 150, 0, true))
    |> Chart.withSize (900.0, 600.0)
    |> Chart.Show
    
createChart(lapPerformance: rawCsv) 
```
The first thing I do is make use of a higher order function, Seq.filter in order to only include lap two for plotting. Thanks to type providers, we can reference columns (e.g. row.Lap) directly while developing! I find this super handy as I no longer need to reference the file itself to figure out which columns are which.

Next, we construct a pipeline using the forward piping operator (\|\>\) to map the lap distance and speed variables as X and Y values on a line chart, and then define how the chart should be styled. I really like the forward piping operator, and I find that it produces some really clean and concise code.

## Exploratory Data with Deedle

For a project at work, I've been exploring the use of [Deedle](https://bluemountaincapital.github.io/Deedle/) as an alternative to Python's Pandas, and it could be a good library to make use of here to explore telemetry data in greater detail.

The first thing I tried using Deedle, was seeing what my average lap speed was across laps 1-4 at the Laguna Seca Racetrack:

```fsharp
let lapCsv = Frame.ReadCsv("""C:\LapTimes.csv""", hasHeaders = true)

let speed = lapCsv.GetColumn<int>("Speed")
speed 
|> Stats.mean
|> printf "Average speed: %A Km/h"
```

The results were:
```
Average speed:
121.036 Km/h
```
In this example, we're loading our CSV file into a Deedle Dataframe, and then binding the Speed column to speed. Next, we make use of pipeline operator to get the mean of the colummn and print that to the console.

## Next Steps

Although there are much better telemetry systems out there, working on my own simple version has allowed me to better see the differences between C# and F#, while at the same time seeing how I can use C# libraries in F# in order to accomplish different tasks. In addition, It would be a good idea to buffer the output as opposed to writing to the file each tick, but since its only two updates per second and doesn't need to scale, I'm okay with leaving it as is for now.

Moving forward, the next step is to parse the CSV file and then plot several laps together to analyze breaking points and speed on the straights to see if I can improve on any specific areas of the track.

Lastly, thanks to F# great interop with C#, I'm not so sure that there's a need to port the library to F#, and I might instead look towards some other areas of F# where there's more of a need for a library.