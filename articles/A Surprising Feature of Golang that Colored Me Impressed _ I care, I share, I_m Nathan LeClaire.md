A Surprising Feature of Golang that Colored Me Impressed | I care, I share, I'm Nathan LeClaire.

# [A Surprising Feature of Golang that Colored Me Impressed](https://nathanleclaire.com/blog/2014/04/27/a-surprising-feature-of-golang-that-colored-me-impressed/)

27 Apr 2014

*EDIT:* Some commenters were confused about some things in this article, and I don’t want people to get an unclear picture, so to clarify:

1. Yes, I know that insertion into a hash table creates an arbitrary ordering of elements by definition. For a variety of reasons, e.g. that not every map is a hash map as some posters have pointed out (and some languages have ordered hash maps), I can see how someone might hypothesize (especially with a naïve understanding of Go maps) that iteration order could be the same as insertion order.

2. My original example was contrived and does not demonstrate the point for most versions of Go (though I hear it might work for 1.3), so I have updated the code to be something you can chuck into an editor or the [Go Playground](http://play.golang.org/p/ppIvkgAGL1) and see the effect for yourself.

3. It *is* true that Go runs from a [random offset for map iteration](https://codereview.appspot.com/5285042/patch/9001/10003). It’s not *just* arbitrary.

Now back to your regularly scheduled article. :)
![](../_resources/bac8929dfa2fc64c865825d9bfc0b505.png)

# go blog.Article()

The amount of enthusiasm and momentum I’ve been seeing regarding the [Go programming language](http://golang.org/) in the past few weeks has been really amazing. Partially this is due to [Gophercon 2014](http://gophercon.com/), which at the time of writing has just occured. I am insanely jealous of the attendees - the format and talks sound like they were awesome, and it’d be great to bump elbows with titans such as [Rob Pike](https://twitter.com/rob_pike) as well as hear all the cool stuff that everyone is building with Go. I feel like additionally I’ve seen a big spike in blog articles related to Go lately, and many are making awesome pivots to include Go in their stack (for instance, [Digital Ocean](https://www.digitalocean.com/company/blog/new-super-fast-droplet-console-thanks-golang/), new cloud startup darling of the masses, just announced they reworked a bunch of Perl code to Go and improved some things such as response time dramatically).

I’ve never written the obligatory “ZOMG I played with Golang for two weeks and it’s awesome” post, since I didn’t really find it had much of a value add in its myriad forms. But recently I came across a Go feature that I considered a very cool reflection of its very excellent (in my opinion) attitude as a language.

Go’s `map` iteration order (using the `range` keyword) is random instead of in the order that the entries were added. What does this mean (context), and why is it significant?

# Maps

## Brief intro to maps

Stolen directly from [*the* article on maps](http://blog.golang.org/go-maps-in-action) by the prolific [Andrew Gerrand](https://twitter.com/enneff):

> One of the most useful data structures in computer science is the hash table. Many hash table implementations exist with varying properties, but in general they offer fast lookups, adds, and deletes. Go provides a built-in map type that implements a hash table.

So in Go if you need a hash table you use a map. Since Go is strongly typed you have to define what type the keys are, and what type the associated values are (e.g. strings, integers, pointers to structs, etc.). A common use case, for instance, it to have a map where the keys are strings and the values they reference are strings.

`m := make(map[string]string)`

Usage is pretty straightforward. Keys don’t need to exist before they are assigned, or even before they are referenced (if they do not exist, we get the value type’s “zero value”).

	m["bandName"] = "Funny Bones"             // "create"
	websiteTitle := m["bandName"] + " Music"  // "read"
	m["bandName"] = "Moon Taxi"               // "update"
	delete(m, "bandName")                     // "delete"
	fmt.Printf(m["bandName"])                 // prints nothing since m["bandName"] == ""

To iterate over all the entries in a map you use the `range` keyword:

	for key, value := range m {
	    fmt.Println("Key:", key, "Value:", value)
	}

## Iteration Order

At first glance a Go programmer might think that the output of this code:

	package main

	import "fmt"

	func main() {
	    blogArticleViews := map[string]int{
	        "unix": 0,
	        "python": 1,
	        "go": 2,
	        "javascript": 3,
	        "testing": 4,
	        "philosophy": 5,
	        "startups": 6,
	        "productivity": 7,
	        "hn": 8,
	        "reddit": 9,
	        "C++": 10,
	    }
	    for key, views := range blogArticleViews {
	        fmt.Println("There are", views, "views for", key)
	    }
	}

Would be this:

	$ go run map_iteration_order.go
	There are 0 views for unix
	There are 1 views for python
	There are 2 views for go
	There are 3 views for javascript
	There are 4 views for testing
	There are 5 views for philosophy
	There are 6 views for startups
	There are 7 views for productivity
	There are 8 views for hn
	There are 9 views for reddit
	There are 10 views for C++

But, since Go 1, the Go runtime actually randomizes the iteration order. So in fact it will be more like this:

	$ go run map_iteration_order.go
	There are 3 views for javascript
	There are 5 views for philosophy
	There are 10 views for C++
	There are 0 views for unix
	There are 1 views for python
	There are 2 views for go
	There are 4 views for testing
	There are 6 views for startups
	There are 7 views for productivity
	There are 8 views for hn
	There are 9 views for reddit

The Go language designers noticed that people were relying on the fact that keys were normally stored in the order they were added in, so they randomized the order in which the keys are iterated over. Thus, if you want to output keys in the order they were added in, you need to keep track of which value is in which position in the order *yourself* like so :

	import "sort"

	var m map[int]string
	var keys []int
	for k := range m {
	    keys = append(keys, k)
	}
	sort.Ints(keys)
	for _, k := range keys {
	    fmt.Println("Key:", k, "Value:", m[k])
	}

Note that the above codeblock is once again shamelessly stolen from [Andrew’s excellent article](http://blog.golang.org/go-maps-in-action).

I think that peoples’ reactions to these sort of things mostly can be categorized into two separate groups.

One group responds with anything from not understanding why this might be something that is done to being slightly peeved to vehemently disapproving. These are most likely the ones who are comfortable making potentially dangerous or magical assumptions about what code is doing behind the scenes and they would prefer that the Go language designers allow them to continue to write dangerous code.

The other group accepts that this was an issue which was addressed, are thankful that the Go language designers are looking out for them, implements the provided solution and moves on.

## Why is it significant?

In one word: attitude.

This seemingly innocuous language feature is something that I consider to be a very good sign in terms of general language philosophy. Instead of trying to be overly flexible and allow sloppy programming, Go forces you to get things straight from the get-go. I think that this is one of the things that contributes to the reported “fuzzy good feeling” that Go programmers reference suggesting that if their program compiles (and especially if it conforms to Go idioms as outlined above), there is a good chance it will work as intended as well. No sneaky typing bugs, missed semi-colons and so on.

In particular Andrew’s referenced article mentions that this was something the Go language designers *changed* rather than continuing to allow people to rely on broken assumptions. One of my hugest pet peeves is when broken or buggy functionality (this could happen in a deliverable, or in a programming language, or elsewhere) becomes a feature through acceptance and workarounds and then a huge stink is raised when the “feature” is attempted to be fixed! It’s pretty clear that, say, PHP and JavaScript have let their culture wander in these directions for various reasons (they’re working on it, but there’s a huge crushing amount of debt to be paid, and some things that will never get fixed).

One of the biggest weak points of PHP, for instance, is the needle-versus-haystack problem. My ideal language (Blub?) would have the sort of attitude that gets driven absolutely crazy by this sort of inconsistency. This is also why I find the Go language designer’s refusal to cave to the cow-towing for exceptions and generics reassuring - they want very badly to *do the right thing* and they know it takes time. They’re in no rush and it’s a lot easier to add features than to un-add them.

# Conclude

Go is a pleasant language and just so well thought-out in many ways. Don’t be too quick to judge or criticize because it lacks features you are accustomed to such as generics or dynamic typing- perhaps if you give it a try you will find that you do not miss them all that much and you are writing simple, clean, elegant code with easy-to-integrate concurrency.

Go is definitely still growing and evolving, and that’s part of the fun of it as well. It is definitely proving to be no less than rock-solid and production-ready, yet still performance and reliability keeps improving. Just check out the awesome numbers on these benchmarks [Rob Pike recently posted](https://groups.google.com/forum/#!msg/golang-dev/2YRmu_AWz68/tKAZgpV7zQwJ) comparing the Go 1 release to tip (nearing 1.3):

Delta from go1 to tip: benchmark old ns/op new ns/op delta BenchmarkBinaryTree17 7102124000 5790215308 -18.47% BenchmarkFannkuch11 7139655000 4361664854 -38.91% BenchmarkFmtFprintfEmpty 177 104 -41.24% BenchmarkFmtFprintfString 575 312 -45.74% BenchmarkFmtFprintfInt 424 230 -45.75% BenchmarkFmtFprintfIntInt 682 403 -40.91% BenchmarkFmtFprintfPrefixedInt 661 394 -40.39% BenchmarkFmtFprintfFloat 907 598 -34.07% BenchmarkFmtManyArgs 2787 1663 -40.33% BenchmarkGobDecode 31284200 10693446 -65.82% BenchmarkGobEncode 13900550 6919498 -50.22% BenchmarkGzip 636714400 704154254 +10.59% BenchmarkGunzip 275620600 139906588 -49.24% BenchmarkHTTPClientServer 144041 71739 -50.20% BenchmarkJSONEncode 83472200 32969241 -60.50% BenchmarkJSONDecode 391968600 120858167 -69.17% BenchmarkMandelbrot200 9540360 6062905 -36.45% BenchmarkGoParse 10007700 6760226 -32.45% BenchmarkRegexpMatchEasy0_32 198 168 -15.15% BenchmarkRegexpMatchEasy0_1K 540 479 -11.30% BenchmarkRegexpMatchEasy1_32 175 149 -14.86% BenchmarkRegexpMatchEasy1_1K 1353 1414 +4.51%BenchmarkRegexpMatchMedium_32 311 307 -1.29% BenchmarkRegexpMatchMedium_1K 108924 126452 +16.09%BenchmarkRegexpMatchHard_32 4972 5681 +14.26%BenchmarkRegexpMatchHard_1K 157354 181042 +15.05%BenchmarkRevcomp 1362067000 1162752845 -14.63% BenchmarkTemplate 714330000 144396424 -79.79% BenchmarkTimeParse 1651 669 -59.48% BenchmarkTimeFormat 3215 714 -77.79%

I love this! And I love Go.

Until next time, stay sassy Internet. And [consider subscribing to my mailing list](https://nathanleclaire.com/).

- Nathan

[Tweet It](https://twitter.com/share?url=https%3a%2f%2fnathanleclaire.com%2fblog%2f2014%2f04%2f27%2fa-surprising-feature-of-golang-that-colored-me-impressed%2f&text=Must-read%20article%20from%20@upthecyberpunks%20:%20A%20Surprising%20Feature%20of%20Golang%20that%20Colored%20Me%20Impressed)[Submit to Hacker News](https://news.ycombinator.com/submitlink?u=https%3a%2f%2fnathanleclaire.com%2fblog%2f2014%2f04%2f27%2fa-surprising-feature-of-golang-that-colored-me-impressed%2f&t=A%20Surprising%20Feature%20of%20Golang%20that%20Colored%20Me%20Impressed)[Submit to Reddit](http://www.reddit.com/submit?url=https%3a%2f%2fnathanleclaire.com%2fblog%2f2014%2f04%2f27%2fa-surprising-feature-of-golang-that-colored-me-impressed%2f&title=A%20Surprising%20Feature%20of%20Golang%20that%20Colored%20Me%20Impressed)