---
layout: guide
title: iRacing Telemetry With F#
tags: [f#, guides, iracing]
header-img: "img/posts/fsharp/fsharp-og-logo.jpg"
---

During the pandemic, I've been spending quite a lot of time playing [iRacing](https://iracing.com) in VR. I love the realism of iRacing (and how well it supports VR!), and managed to score a win in the Mazda MX-5 this season at the now defunct Oran Park raceway. 

<img src="/img/posts/race-result.png" width="750" height="211" alt="First place at Oran Park Raceway">

In a [previous post](https://markjames.dev/2021-01-08-writing-an-iracing-sdk-implementation-fsharp/), I discussed how I was looking to create an F# library for the iRacing SDK. This proved to be a bigger challenge than I thought as the iRacing API uses a memory mapped file and I'm not yet familiar enough with F# to be tackling the problem yet.

As an alternative (and thanks to F#'s interoperability with C# and the .NET platform), I decided to familiarize myself with this [C# iRacing SDK](https://github.com/NickThissen/iRacingSdkWrapper) and build a small app that gathers some basic telemetry data and writes it to a CSV file which I could then plot in a .NET Interactive Notebook using Plotly.NET. The end result of my first few tests looked like this:

<div id="2ab460b2-04c6-45dc-abcd-35245dddf90c" style="width: 700px; height: 350px;"><!-- Plotly chart will be drawn inside this DIV --></div>
<script type="text/javascript">

            var renderPlotly_2ab460b204c645dcabcd35245dddf90c = function() {
            var fsharpPlotlyRequire = requirejs.config({context:'fsharp-plotly',paths:{plotly:'https://cdn.plot.ly/plotly-latest.min'}}) || require;
            fsharpPlotlyRequire(['plotly'], function(Plotly) {

            var data = [{"type":"scatter","x":[0.0,0.01,0.02,0.03,0.03,0.04,0.05,0.06,0.07,0.08,0.09,0.09,0.1,0.11,0.11,0.12,0.12,0.13,0.13,0.13,0.14,0.14,0.14,0.15,0.15,0.15,0.16,0.16,0.17,0.17,0.18,0.18,0.19,0.19,0.2,0.21,0.21,0.22,0.22,0.23,0.23,0.23,0.24,0.24,0.25,0.26,0.26,0.27,0.27,0.28,0.29,0.29,0.3,0.3,0.31,0.31,0.32,0.33,0.33,0.34,0.35,0.36,0.36,0.37,0.38,0.39,0.4,0.4,0.41,0.42,0.42,0.42,0.43,0.43,0.44,0.45,0.45,0.46,0.46,0.47,0.48,0.48,0.49,0.5,0.5,0.51,0.52,0.53,0.53,0.54,0.54,0.55,0.55,0.56,0.57,0.57,0.58,0.59,0.59,0.6,0.61,0.62,0.62,0.63,0.64,0.65,0.66,0.66],"y":[193,197,200,203,206,210,213,217,220,224,211,188,167,147,127,105,90,88,85,82,88,96,98,93,89,84,88,102,114,123,132,141,149,154,150,145,131,124,119,114,114,119,126,136,144,151,158,159,144,141,139,138,138,145,152,159,166,172,178,184,188,193,197,202,206,209,188,161,132,121,117,116,120,125,131,137,143,150,156,163,168,173,178,183,186,189,171,148,140,137,139,146,153,159,165,170,174,178,181,183,186,189,193,196,199,196,174,152],"mode":"lines","line":{"width":{},"shape":"spline"},"marker":{}}];
            var layout = {"title":"Laguna Seca (Ferrari 488 GTE)","xaxis":{"title":"Lap Distance (%)","showgrid":false,"position":200.0},"yaxis":{"title":"Speed (Km/h)","showgrid":false},"margin":{"l":120,"r":100,"t":50,"b":150,"pad":0,"autoexpand":true},"width":700.0,"height"350.0};
            var config = {};
            Plotly.newPlot('2ab460b2-04c6-45dc-abcd-35245dddf90c', data, layout, config);
});
            };
            if ((typeof(requirejs) !==  typeof(Function)) || (typeof(requirejs.config) !== typeof(Function))) {
                var script = document.createElement("script");
                script.setAttribute("src", "https://cdnjs.cloudflare.com/ajax/libs/require.js/2.3.6/require.min.js");
                script.onload = function(){
                    renderPlotly_2ab460b204c645dcabcd35245dddf90c();
                };
                document.getElementsByTagName("head")[0].appendChild(script);
            }
            else {
                renderPlotly_2ab460b204c645dcabcd35245dddf90c();
            }
</script>

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

private 
```

In F#, we subscribe to the event using Observable.subscribe and call our writeToCsv function each time the event is fired:
```fsharp
    |> Observable.subscribe writeToCsv |> ignore
```

The interop between C# and F# events is nice, and I find this solution to be quite elegant. In addition, you also have Observable.scan which accumulates state each time an event has fired. I haven't had to do that in this example, but you can see [more details here](https://fsharpforfunandprofit.com/posts/concurrency-reactive/).

## Plotting the Telemetry Data

Now that we have a backend system logging some telemetry data from the sim, the next step is to plot the graph we saw above. In order to do so, I created a new .NET Interactive notebook in VS Code Insiders. I then imported the libraries I needed and wrote the following intitial code:
```fsharp
[<Literal>]
let FilePath = """C:\LapTimes.csv"""

type rawCsv = CsvProvider<FilePath, HasHeaders = true>

// CSV File
let lapPerformance = rawCsv.GetSample()
```
The use of [<Literal>] here is because FilePath must be a constant so that the CsvProvider can read the data while we're developing. I also added some column headers into my CSV file (in the future, I plan to write these automatically in the backend code). We also create a new CsvProvider, and then get the data by calling GetSample() on the raw CSV file.

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

Next, we construct a pipeline using the forward piping operator (|>) to map the lap distance and speed vartiables as X and Y values on a line chart, and then define how the chart should be styled. I really like the forward piping operator, and I find that it produces some really clean and concise code.

## Other Exploratory Data

Another thing I tried was seeing what my average lap speed was across laps 1-4:

```fsharp
let lapCsv = Frame.ReadCsv("""C:\LapTimes.csv""", hasHeaders = true)

let speed = lapCsv.GetColumn<int>("Speed")
speed |> Stats.mean
|> printf "Average speed: %A"
```

The results were:
```
Average speed:
121.0335937
```
## Next Steps

Although there are much better telemetry systems out there, working on my own simple version has allowed me to better see the differences between C# and F#, while at the same time seeing how I can use C# libraries in F# in order to accomplish different tasks.

Moving forward, the next step is to parse the CSV file and then plot several laps together to analyze breaking points and speed on the straights to see if I can improve on any specific areas of the track.

Lastly, It would also be a good idea to buffer the output as opposed to writing to the file each tick, but since its only 2 updates per second and doesn't need to scale, I'm okay with leaving it as is for now.