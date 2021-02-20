---
layout: guide
title: Drawing a Racetrack From a CSV File With C#
header-img: "img/posts/opengraph/csharp-logo.png"
tags: [guides, programming projects, c#] 
---

Although 2021 has been the year of [learning F#](/2021-01-04-why-learning-fsharp-2021/) for me, I was recently approached by a company looking to have me do some part time work in C#. As I find the work they're doing to be quite interesting, I've spent this chilly Saturday in February working on a small project in C# to get myself back into the groove of things before diving into the work in a week or so.

I'm a big fan of Google maps (I've used it extensively for years to explore all kinds of places!), and recently I've become interested in the Open Street Map (OSM) project and the work being done on it. While browsing for some OSM related projects, I happened to come across this [Github repo](https://github.com/TUMFTM/racetrack-database) which contains center lines, track widths, and race lines for a variety of automotive racetracks across the world. According to the readme file, The original center lines were fetched as GPS points from the OpenStreetMap project, and the author applied a smoothing algorithm to the center lines of the track.

As I've mentioned in a few previous articles, I've been doing quite a lot of experimenting with [iRacing telemetry](https://markjames.dev/2021-02-09-iracing-telemetry-fsharp/) lately. In keeping with the theme of racing, I decided to create a small program which creates a PNG map of my hometown Formula 1 circuit, Circuit Gilles Villeneuve in Montreal. The end result looked like this:

<img src="/img/posts/gilles-villeneuve-trackmap-cropped.png" width="375" height="1065" alt="The Gilles Villeneuve Racetrack created from GPS coordinates">

## Parsing the CSV File with CSVHelper

The first step of the project was to download the track data for Montreal [here](https://github.com/TUMFTM/racetrack-database/blob/master/tracks/Montreal.csv). Upon inspecting the file, it contained four columns; the center lines (x, y), and the track widths to the right (w_tr_right) and left (w_tr_left). As I wanted to keep the project fairly simple, I only needed to make use of the X Y center lines which I could then plot.

In order to parse the data, I grabbed the terrific [CsvHelper](https://joshclose.github.io/CsvHelper/getting-started) library from NuGet. Next, I created a new Track class in order to model the data that I needed from the Csv file:

```csharp
class Track
{
    public float X { get; set; }
    public float Y { get; set; }
}
```
I also edited the column headers of the X & Y coordinates to make things more readable:

```
X,Y,w_tr_right_m,w_tr_left_m
```

With the track class in place, the next step was to load the Csv file using StreamReader and parse it with CsvReader. In doing so, I made use of simplified using statements (introduced in C# 8) to load the file inside the program's entry point:

```csharp
static void Main(string[] args)
{
    using var reader = new StreamReader(@"C:\Tracks\Montreal.csv");
    using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

    // Get the XY values of track points from CSV file.
    var trackValues = csv.GetRecords<Track>();
}
```
The resulting trackValues variable is an IEnumerable of type Track which would form the basis for plotting the track.

## Plotting the Track Using System.Drawing

The [System.Drawing](https://docs.microsoft.com/en-us/dotnet/api/system.drawing?view=dotnet-plat-ext-5.0) namespace by Microsoft provides access to GDI+ graphics functionality. Making use of System.Drawing would enable me to create and export the resulting track plot to a PNG file.

Since the System.Drawlines() method requires the use of an array of Points to draw an image, the first thing I needed to do was to create a list to store the GPS coordinates, add the coordinates, and then convert that list to an array of Points. In C#, you cannot change the array size once is created, and so its easier to start with a list and then convert it using the ToArray() method. Below my trackValues variable, I added the following lines of code:

```csharp
// Create a list to store our points.
List<PointF> points = new List<PointF>();

// Create the XY coords as PointF and add it to our list. 
foreach (Track track in trackValues)
{
    var pt = new PointF(track.X, track.Y);
    points.Add(pt);
 }

// Convert the points list to an array for Drawlines(). 
PointF[] pointArray = points.ToArray();
```
Note that as the GPS coordinates are floats, I'm using PointF as opposed to Point here.

With an array filled with our GPS coordinates in the proper format, the next step was to setup the look and feel of the image that we'll be exporting. Below the above code, I added in the following lines:
```csharp
// Define the image style.
using var bmp = new Bitmap(1300, 2200);
using var gfx = Graphics.FromImage(bmp);
using var pen = new Pen(Color.Black, 20);

// Smooth the plot using AntiAliasing and create a white background.
gfx.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;
gfx.Clear(Color.White);

// Change the coordinate system so that the track is centered in the image.
gfx.TranslateTransform(900, 500);
```
Although this code is mostly self explanatory, note the use of TranslateTransform here. Originally I had omitted this line, and kept running into an issue where I could only see a tiny bit of the plot while doing some test renders. This is because the view defaults to (0,0) and most of the GPS coordinates are in the negative. By using TranslateTransform(), I was able to change the origin of the coordinate system by prepending the specified translation to the transformation matrix of this System.Drawing.Graphics.

The final step was to draw the racetrack and save the resulting image to a file like so:
```csharp
// Draw the track & save the resulting image as a PNG.
gfx.DrawLines(pen, pointArray);
bmp.Save(@"C:\Tracks\GillesVilleneuve.png");
Console.WriteLine("The track image has been saved successfully.");
```
Success!

## Conclusion 

The process of reading a CSV file of GPS coordinates and then drawing them and saving to a file was a painless process in C#, and this was a fun project to get my brain used to the syntax of C#  after having spent the past few months working in F#. Going forward, it could be interesting to take into account the track widths and plot the ideal racing line inside of them, but that's a project for another day! 