---
layout: post
title: Making the Best of Physical Distancing
---
With Canada practicing aggressive physical distancing due to the ongoing pandemic, I’ve been finding myself with a lot of free time indoors lately which I’ve been devoting towards a few different personal projects. Just yesterday, I finished updating my iOS app [CU Libraries](https://markjames.dev/cu-libraries) to be compliant with the latest version of iOS 13. After the latest iOS update, I had noticed that the app UI wouldn’t properly render on devices with a notch (like my iPhone XS Max). After doing some debugging, I discovered a fix which involved commenting out a single line of code: 

<code>.edgesIgnoringSafeArea(.top)</code>

This code tells SwiftUI to ignore the safe area (the area around the notch of your device), which was a workaround I used to render the top indigo section of my CU Libraries UI here:

<center><img src="https://user-images.githubusercontent.com/20845425/79627392-cfd6c580-8105-11ea-9c5d-38349580bba2.png" width="520"/></center>

However, in the latest version of iOS, It seems that area around the notch is now handled automatically by SwiftUI, and if your app code ignores the safe area it won’t render properly, but only on notch devices (using the above code worked fine running on the iPhone 8 simulator). This was the first update I’ve pushed to the first app I’ve ever published on the App Store, and so I was pretty excited when I received an email from Apple earlier today approving the update! 

Aside from the app update, I’ve also been making extensive use of Swift Playgrounds. Playgrounds have been an awesome way to make quick conceptual sketches which I find great for iterative learning. For example, after learning about protocols in Swift, I was quickly able to create a Playground which let me test out how to use protocols effectively in a range of different scenarios that I could play around with in real time. The playgrounds also proved really useful in helping me [model a deck of cards](https://gist.github.com/markjamesm/08bf727b1113d4989e59f7073697dc17) using a collection of enums and structs for the first time (next step is implementing blackjack functionality! 

Now that I have the console based playgrounds under my belt, the next step is exploring how to combine SwiftUI with playgrounds over the weekend. With Playgrounds, I’m excited to be able to rapidly prototype my code prior to building it out in a full XCode project.

In between coding sessions, I’ve been practicing learning bass guitar, which has been going well since the note layout of the bass is the same as the [Linnstrument](https://www.rogerlinndesign.com/linnstrument) which I’ve gotten quite proficient at playing. I’ve been wanting to pick up bass for a while now but just haven’t had the time to practice my technique. Between the Bass and Linnstrument, I have a few different ways to creatively express myself in between learning sessions. Interestingly, I find the music really helps my coding. The more I practice, the easier I find coding sessions become. It seems like when I’m playing some improv jazz or playing in the groove it really helps to keep me sharp and able to focus on note taking and coding challenges.

I’ve also been careful to limit my time reading the news and have almost entirely cut out social media. When I’m home in my apartment, I find spending too much time on these types of sites can really eat into my productivity. However, being aware of something being an issue is half the battle, and so I’ve done my best to avoid getting into a habit of checking these kinds of sites more than once a day.

I hope everyone is keeping safe, and if you find yourself getting bored during physical distancing, I highly encourage you to take some time to dedicate to activities that you might not normally have the time to explore!
