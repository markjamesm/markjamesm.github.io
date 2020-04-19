---
layout: post
title: Building a dead link checker in Swift to analyze my bookmarks
header-img: "img/posts/link-checker.png"
tags: [swift, programming projects, tutorials]
---

Having spent the past few months devoting a significant amount of time learning to code with Apple's Swift, I'm slowly becoming more and more confident in my abilities to implement ideas in code. Much like music production, the programming learning curve can be very steep at the start, but as you keep at it and take things day by day, you start to see real results. As I continue to learn through a range of sources including LinkedIn Learning and [SwiftBySundell](https://swiftbysundell.com), I'm always looking to try new things to expand my burgeoning skillset. Fortunately, just the other day an interesting experiment popped into my head. While perusing Wikipedia like I often do, I came across a topic on [link rot](https://en.wikipedia.org/wiki/Link_rot) which piqued my interest. 

For roughly twelve years now, I've been collecting and saving bookmarks on my browsers and keeping them synced across my computers, and as I read the wiki page, I started thinking how it would be interesting to create a broken link checker for bookmarks in Swift and documenting the process. According to the program I ended up writing called [Bookmark Tester](https://github.com/markjamesm/bookmark-tester), I have 1,078 bookmarks and am eager to see how many of them are still live!

The first step in creating Bookmark Tester was to export my bookmarks as an HTML file, which seems to be the only export option supported in Chrome. For the sake of example code in this article, I'm going to be working with an example bookmarks file ([sourcecode](https://markjames.dev/samplebookmarks.html)) that I created. The link analysis comes from my actual collection of bookmarks however. 

Looking though the HTML, I could see the bookmark links nested inside a description list as description tags along with their corresponding favicons and descriptions, with each bookmark entry looking something like this:

 ```
 <DT><A HREF="https://www.wikipedia.org/" ADD_DATE="1587262550" ICON="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAABO0lEQVQ4jaWTMaoCMRCG/wnvDtELmHaxdAmIXcheZA9hYeMNxNZqsc81lu0X+2VLTzBj8V5C8uQ9UAcG5k+YP5kvhPATzCx4IZRSBAD0TnNuQu82J5NPmgFADcMAay2UUjifzwAA733S8zzDWgtrLeZ5xvV6xXK5hPcet9vte/5pmoSIJIQgURtj5HQ6CTOLc06maRJmFmaWuq5TjVg454qNrutEay0hBDkej8V6NC4M+r4XANL3fdo0xogxJul4UK4TxPV6Decc9vt9ArTb7XC/35MehgFVVZUUc7cQghCRjOOYTtNaS9d1wszStm3BgpnlKzfz3mO1WuFyuWCz2aBpGlhrcTgcsN1uAQCLxeLvG0RIRJRmjS9U13XB5wlinlrrgnTbtk/w/jWIDPL8PXvMzz9TzuLVZgB4AExRsO8ga8hoAAAAAElFTkSuQmCC">Wikipedia</A>
```
 
 Since I only needed to grab the URLs and nothing else, the next step was to create a new XCode project and use an HTML parsing library to isolate the bookmark URLs. After some cursory Googling, I found [SwiftSoup](https://github.com/scinfu/SwiftSoup) which looked to be the perfect parsing solution. After a quick and painless install using the Swift Package Manager, I started thinking about how I would grab the HTMLs. 
 
 First, I created two structs, BrowserBookmarks and BookmarkParser. The BrowserBookmarks struct was simple enough and had only one element, a string variable containing my entire bookmarks file. This string can be ingested by the SwiftSoup parser, which is exactly what happens inside BrowserParser's getBookmarksLinks() method, which consists of the following code:
 
     mutating func getBookmarkLinks() {
             
        let html = BrowserBookmarks().bookmarkHTML
         
        guard let bookmarks: Elements = try? SwiftSoup.parse(html).select("a") else { return }
        for bookmarkLink: Element in bookmarks.array() {
           let cleanLink = try? bookmarkLink.attr("href")
           self.bookmarkURLS.insert(cleanLink ?? "Error")
        }
     }   

In this code, I get each URL and then store it in a set. To make sure the function worked as intended, I created a simple method called listBookmarks() to print each element in the set. Calling this method on my test bookmarks yielded the following console output:

````
https://news.ycombinator.com/ffdbfbbd
https://news.ycombinator.com/
https://www.youtube.com/
https://developer.apple.com/swift/
https://www.wikipedia.org/
https://www.google.com/
http://brokenwebsitelink.com/

Program ended with exit code: 0
````

Everything working so far! In case you haven't tried each of the links, some of them are broken. This was done intentionally in order to test the link checking method in the next step.

With a set of clean bookmark URLs, the next step was to create the checkLink() method, which takes one parameter (a URL) and checks the link by using URLSession to download the response codes. I put this method inside a class called LinkChecker, to which I also added three properties which keep track of the clean, broken and unknown link statuses. In order to give URLSession enough time to properly download the HTTP headers, Because we need to wait for the URL session task to finish downloading the response codes, I made use of semaphores to start and stop the code from executing at the right points. When I finished writing the method, this is what I had:

```
func checkLink(link: String) {
           
   // Create a semaphore to wait until URLSession finishes downloading
   let semaphore = DispatchSemaphore( value: 0)
           
    let request = URLRequest(url: URL(string: link)!)
    URLSession.shared.dataTask(with: request) { (data, response, error) in
    semaphore.signal()
    
    // Check if the response has an error
    if error != nil{
        self.brokenLinks += 1
        return
        }

    if let httpResponse = response as? HTTPURLResponse{
               
         // Get the status codes and increment the appropriate counter
         if httpResponse.statusCode == 404 {
            self.brokenLinks += 1
         }
                  
         if httpResponse.statusCode == 200 {
            self.cleanLinks += 1
         }
       }
    }.resume()
  semaphore.wait()     
 }
```

I then tested the method on the test bookmarks file I created earlier, using the following code in my main.swift file:

``` 
var parser = BookmarkParser()
var checker = LinkChecker()

parser.getBookmarkLinks()

for bookmark in parser.bookmarkURLS {
    checker.checkLink(link: bookmark)
}

print("Clean Links: \(checker.cleanLinks)")
print("Broken Links: \(checker.brokenLinks)")
print("Dead Links: \(checker.deadLinks)")
```
Sure enough, the results were right on the money!

```
Clean Links: 5
Broken Links: 2
Dead Links: 1
```
There were a total of 8 test links in my [sample bookmarks file](https://markjames.dev/samplebookmarks.html), and the link checker successfully categorized each one! 

With the program now operational, the final step was to run my full list of bookmarks through and see what happened. With 1078 bookmarks, I expected the process to take a little while and so to add some verbosity, I added the following line to my main.swift for loop:

``
    print("checking \(bookmark)")
``

Now, as each bookmark is being checked, I can see status messages like these: 

```
checking https://developer.apple.com/swift/
checking https://news.ycombinator.com/ffdbfbbd
checking https://news.ycombinator.com/
```

However after running my full bookmarks list for the first time, my program crashed about halfway through the list of URLs to check. Some debugging showed me that this line of code was the culprit:

```
let request = URLRequest(url: URL(string: link)!)
```

By force unwrapping the URLRequest, my code was throwing an error when it came across a nil value in my list of parsed URLs. I suspect that this is due to some of the bookmark URLs having weird characters in them. Fortunately, I was able to solve the problem by making use of a guard statement:

```
guard let url = URL(string: link) else { return }
let request = URLRequest(url: url)
URLSession.shared.dataTask(with: request) { (data, response, error) in
```

With the guard statement in place, my code no longer crashed on nil and I learned why force unwrapping optionals is a bad idea in the process!

Running my full list of bookmarks through the checker returned the following results with % of totals added in:

```
Clean Links: 718
Broken Links: 196
Dead Links: 122
```

Putting that data into a graph, here's a breakdown of the status of my bookmarks: 

<center><img src="https://user-images.githubusercontent.com/20845425/79681463-76939280-81e8-11ea-9b77-2c5b6ed534f9.png" /></center>

Out of the 1,078 bookmarks I exported, the parser was able to successfully read and analyze 1,036 of them, which is a great sampling. 30.7% of those bookmarks are no longer functional, with 18,9% of them being broken and 11.8% completely dead. As I mentioned earlier, these bookmarks date start from about 2008 until the present, although these days, thanks to other solutions I save a lot less traditional bookmarks than I used to. In fact, I think I've bookmarked maybe 5 links in the past year. Aside from the coding element, its an interesting exercise seeing how much content churn happens on the web. When I think back to how many websites I've created and then taken down for various reasons over the years, it does make sense how this can happen.

This project was a lot of fun, and it was interesting to finally figure out an approximate count of how many dead bookmarks I've been carrying around all these years. Although the link checker works, its still quite rudimentary and could use some additional fleshing out (possibly as a future project). For example, I'm only handling a very narrow range of the most common status codes, and its very possible that some servers won't send a response code to a scraper bot. Despite this, it felt really empowering being able to quickly implement an idea in code in several hours, whereas a project like this would have taken me several days in the past!   

If you're curious to see the finished project (or want to fork it!), you can find it [here on Github!](https://github.com/markjamesm/bookmark-tester)