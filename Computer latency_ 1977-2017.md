Computer latency: 1977-2017

 **Computer latency: 1977-2017**
* * *

I have this nagging feeling that the computers I use today feel slower than the computers I used as a kid. I don’t trust this kind of feeling because human perception has been shown to be unreliable in empirical studies, so I carried around a high-speed camera and measured the response latency of devices I’ve run into in the past few months. Here are the results:

| computer | latency<br>(ms) | year | clock | # T |
| --- | --- | --- | --- | --- |
| apple 2e | 30  | 1983 | 1 MHz | 3.5k |
| ti 99/4a | 40  | 1981 | 3 MHz | 8k  |
| custom haswell-e *165Hz* | 50  | 2014 | 3.5 GHz | 2G  |
| commodore pet 4016 | 60  | 1977 | 1 MHz | 3.5k |
| sgi indy | 60  | 1993 | .1 GHz | 1.2M |
| custom haswell-e *120Hz* | 60  | 2014 | 3.5 GHz | 2G  |
| thinkpad 13 **chromeos** | 70  | 2017 | 2.3 GHz | 1G  |
| imac g4 **os 9** | 70  | 2002 | .8 GHz | 11M |
| custom haswell-e *60Hz* | 80  | 2014 | 3.5 GHz | 2G  |
| mac color classic | 90  | 1993 | 16 MHz | 273k |
| powerspec g405 **linux**  *60Hz* | 90  | 2017 | 4.2 GHz | 2G  |
| macbook pro 2014 | 100 | 2014 | 2.6 GHz | 700M |
| thinkpad 13 **linux chroot** | 100 | 2017 | 2.3 GHz | 1G  |
| lenovo x1 carbon 4g **linux** | 110 | 2016 | 2.6 GHz | 1G  |
| imac g4 **os x** | 120 | 2002 | .8 GHz | 11M |
| custom haswell-e *24Hz* | 140 | 2014 | 3.5 GHz | 2G  |
| lenovo x1 carbon 4g **win** | 150 | 2016 | 2.6 GHz | 1G  |
| next cube | 150 | 1988 | 25 MHz | 1.2M |
| powerspec g405 **linux** | 170 | 2017 | 4.2 GHz | 2G  |
| packet around the world | 190 |
| powerspec g405 **win** | 200 | 2017 | 4.2 GHz | 2G  |
| symbolics 3620 | 300 | 1986 | 5 MHz | 390k |

These are tests of the latency between a keypress and the display of a character in a terminal (see appendix for more details). The results are sorted from quickest to slowest. In the latency column, the background goes from green to yellow to red to black as devices get slower and the background gets darker as devices get slower. No devices are green. When multiple OSes were tested on the same machine, the os is **in bold**. When multiple refresh rates were tested on the same machine, the refresh rate is *in italics*.

In the year column, the background gets darker and purple-er as devices get older. If older devices were slower, we’d see the year column get darker as we read down the chart.

The next two columns show the clock speed and number of transistors in the processor. Smaller numbers are darker and blue-er. As above, if slower clocked and smaller chips correlated with longer latency, the columns would get darker as we go down the table, but it, if anything, seems to be the other way around.

For reference, the latency of a packet going around the world through fiber from NYC back to NYC via [Tokyo and London](https://www.extremetech.com/extreme/122989-1-5-billion-the-cost-of-cutting-london-toyko-latency-by-60ms) is inserted in the table.

If we look at overall results, the fastest machines are ancient. Newer machines are all over the place. Fancy gaming rigs with unusually high refresh-rate displays are almost competitive with machines from the late 70s and early 80s, but “normal” modern computers can’t compete with thirty to forty year old machines.

We can also look at mobile devices. In this case, we’ll look at scroll latency in the browser:

| device | latency<br>(ms) | year |
| --- | --- | --- |
| ipad pro 10.5" pencil | 30  | 2017 |
| ipad pro 10.5" | 70  | 2017 |
| iphone 4s | 70  | 2011 |
| iphone 6s | 70  | 2015 |
| iphone 3gs | 70  | 2009 |
| iphone x | 80  | 2017 |
| iphone 7 | 80  | 2017 |
| iphone 6 | 80  | 2014 |
| gameboy color | 80  | 1989 |
| iphone 5 | 90  | 2012 |
| blackberry q10 | 100 | 2013 |
| huawei honor 8 | 110 | 2016 |
| google pixel 2 xl | 110 | 2017 |
| galaxy s7 | 120 | 2016 |
| galaxy note 3 | 120 | 2016 |
| nexus 5x | 120 | 2015 |
| oneplus 3t | 130 | 2016 |
| blackberry key one | 130 | 2017 |
| moto e (2g) | 140 | 2015 |
| moto g4 play | 140 | 2017 |
| moto g4 plus | 140 | 2016 |
| google pixel | 140 | 2016 |
| samsung galaxy avant | 150 | 2014 |
| asus zenfone3 max | 150 | 2016 |
| sony xperia z5 compact | 150 | 2015 |
| htc one m4 | 160 | 2013 |
| galaxy s4 mini | 170 | 2013 |
| lg k4 | 180 | 2016 |
| packet | 190 |
| htc rezound | 240 | 2011 |
| palm pilot 1000 | 490 | 1996 |
| kindle paperwhite 3 | 630 | 2015 |
| kindle 4 | 860 | 2011 |

As above, the results are sorted by latency and color-coded from green to yellow to red to black as devices get slower. Also as above, the year gets purple-er (and darker) as the device gets older.

If we exclude the `game boy color`, which is a different class of device than the rest, all of the quickest devices are Apple phones or tablets. The next quickest device is the `blackberry q10`. Although we don’t have enough data to really tell why the `blackberry q10` is unusually quick for a non-Apple device, one plausible guess is that it’s helped by having actual buttons, which are easier to implement with low latency than a touchscreen. The other two devices with actual buttons are the `gameboy color` and the `kindle 4`.

After that `iphones` and non-kindle button devices, we have a variety of Android devices of various ages. At the bottom, we have the ancient `palm pilot 1000` followed by the kindles. The `palm` is hamstrung by a touchscreen and display created in an era with much slower touchscreen technology and the `kindles` use [e-ink](https://en.wikipedia.org/wiki/E_Ink) displays, which are much slower than the displays used on modern phones, so it’s not surprising to see those devices at the bottom.

### Why is the `apple 2e` so fast?

Compared to a modern computer that’s not the latest `ipad pro`, the `apple 2` has significant advantages on both the input and the output, and it also has an advantage between the input and the output for all but the most carefully written code since the `apple 2` doesn’t have to deal with context switches, buffers involved in handoffs between different processes, etc.

On the input, if we look at modern keyboards, it’s common to see them scan their inputs at `100 Hz` to `200 Hz` (e.g., [the ergodox claims to scan at `167 Hz`](https://github.com/benblazak/ergodox-firmware)). By comparison, the `apple 2e` effectively scans at `556 Hz`. See appendix for details.

If we look at the other end of the pipeline, the display, we can also find latency bloat there. I have a display that advertises `1 ms` switching on the box, but if we look at how long it takes for the display to actually show a character from when you can first see the trace of it on the screen until the character is solid, it can easily be `10 ms`. You can even see this effect with some high-refresh-rate displays that are sold on their allegedly good latency.

At `144 Hz`, each frame takes `7 ms`. A change to the screen will have `0 ms` to `7 ms` of extra latency as it waits for the next frame boundary before getting rendered (on average,we expect half of the maximum latency, or `3.5 ms`). On top of that, even though my display at home advertises a `1 ms` switching time, it actually appears to take `10 ms` to fully change color once the display has started changing color. When we add up the latency from waiting for the next frame to the latency of an actual color change, we get an expected latency of `7/2 + 10 = 13.5ms`

With the old CRT in the `apple 2e`, we’d expect half of a `60 Hz` refresh (`16.7 ms / 2`) plus a negligible delay, or `8.3 ms`. That’s hard to beat today: a state of the art “gaming monitor” can get the total display latency down into the same range, but in terms of marketshare, very few people have such displays, and even displays that are advertised as being fast aren’t always actually fast.

### iOS rendering pipeline

If we look at what’s happening between the input and the output, the differences between a modern system and an `apple 2e` are too many to describe without writing an entire book. To get a sense of the situation in modern machines, here’s former iOS/UIKit engineer [Andy Matuschak](https://andymatuschak.org/)’s high-level sketch of what happens on iOS, which he says should be presented with the disclaimer that “this is my out of date memory of out of date information”:

- hardware has its own scanrate (e.g. `120 Hz` for recent touch panels), so that can introduce up to `8 ms` latency

- events are delivered to the kernel through firmware; this is relatively quick but system scheduling concerns may introduce a couple `ms` here

- the kernel delivers those events to privileged subscribers (here, `backboardd`) over a mach port; more scheduling loss possible

- `backboardd` must determine which process should receive the event; this requires taking a lock against the window server, which shares that information (a trip back into the kernel, more scheduling delay)

- `backboardd` sends that event to the process in question; more scheduling delay possible before it is processed

- those events are only dequeued on the main thread; something else may be happening on the main thread (e.g. as result of a timer or network activity), so some more latency may result, depending on that work

- UIKit introduced `1-2 ms` event processing overhead, CPU-bound

- application decides what to do with the event; apps are poorly written, so usually this takes many `ms`. the consequences are batched up in a data-driven update which is sent to the render server over IPC

    - If the app needs a new shared-memory video buffer as a consequence of the event, which will happen anytime something non-trivial is happening, that will require round-trip IPC to the render server; more scheduling delays

    - (trivial changes are things which the render server can incorporate itself, like affine transformation changes or color changes to layers; non-trivial changes include anything that has to do with text, most raster and vector operations)

    - These kinds of updates often end up being triple-buffered: the GPU might be using one buffer to render right now; the render server might have another buffer queued up for its next frame; and you want to draw into another. More (cross-process) locking here; more trips into kernel-land.

- the render server applies those updates to its render tree (a few `ms`)

- every `N Hz`, the render tree is flushed to the GPU, which is asked to fill a video buffer

    - Actually, though, there’s often triple-buffering for the screen buffer, for the same reason I described above: the GPU’s drawing into one now; another might be being read from in preparation for another frame

- every `N Hz`, that video buffer is swapped with another video buffer, and the display is driven directly from that memory

    - (this `N Hz` isn’t necessarily ideally aligned with the preceding step’s `N Hz`)

Andy says “the actual amount of *work* happening here is typically quite small. A few `ms` of CPU time. Key overhead comes from:”

- periodic scanrates (input device, render server, display) imperfectly aligned

- many handoffs across process boundaries, each an opportunity for something else to get scheduled instead of the consequences of the input event

- lots of locking, especially across process boundaries, necessitating trips into kernel-land

By comparison, on the Apple 2e, there basically aren’t handoffs, locks, or process boundaries. Some very simple code runs and writes the result to the display memory, which causes the display to get updated on the next scan.

### Refresh rate vs. latency

One thing that’s curious about the computer results is the impact of refresh rate. We get a `90 ms` improvement from going from `24 Hz` to `165 Hz`. At `24 Hz` each frame takes `41.67 ms` and at `165 Hz` each frame takes `6.061 ms`. As we saw above, if there weren’t any buffering, we’d expect the average latency added by frame refreshes to be `20.8ms` in the former case and `3.03 ms` in the latter case (because we’d expect to arrive at a uniform random point in the frame and have to wait between `0ms` and the full frame time), which is a different of about `18ms`. But the difference is actually `90 ms`, implying we have latency equivalent to `(90 - 18) / (41.67 - 6.061) = 2` buffered frames.

If we plot the results from the other refresh rates on the same machine (not shown), we can see that they’re roughly in line with a “best fit” curve that we get if we assume that, for that machine running powershell, we get 2.5 frames worth of latency regardless of refresh rate. This lets us estimate what the latency would be if we equipped this low latency gaming machine with an `infinity Hz` display – we’d expect latency to be `140 - 2.5 * 41.67 = 36 ms`, almost as fast as quick but standard machines from the 70s and 80s.

### Complexity

Almost every computer and mobile device that people buy today is slower than common models of computers from the 70s and 80s. Low-latency gaming desktops and the `ipad pro` can get into the same range as quick machines from thirty to forty years ago, but most off-the-shelf devices aren’t even close.

If we had to pick one root cause of latency bloat, we might say that it’s because of “complexity”. Of course, we all know that complexity is bad. If you’ve been to a non-academic non-enterprise tech conference in the past decade, there’s a good chance that there was at least one talk on how complexity is the root of all evil and we should aspire to reduce complexity.

Unfortunately, it’s a lot harder to remove complexity than to give a talk saying that we should remove complexity. A lot of the complexity buys us something, either directly or indirectly. When we looked at the input of a fancy modern keyboard vs. the `apple 2` keyboard, we saw that using a relatively powerful and expensive general purpose processor to handle keyboard inputs can be slower than dedicated logic for the keyboard, which would both be simpler and cheaper. However, using the processor gives people the ability to easily customize the keyboard, and also pushes the problem of “programming” the keyboard from hardware into software, which reduces the cost of making the keyboard. The more expensive chip increases the manufacturing cost, but considering how much of the cost of these small-batch artisanal keyboards is the design cost, it seems like a net win to trade manufacturing cost for ease of programming.

We see this kind of tradeoff in every part of the pipeline. One of the biggest examples of this is the OS you might run on a modern desktop vs. the loop that’s running on the `apple 2`. Modern OSes let programmers write generic code that can deal with having other programs simultaneously running on the same machine, and do so with pretty reasonable general performance, but we pay a huge complexity cost for this and the handoffs involved in making this easy result in a significant latency penalty.

A lot of the complexity might be called [accidental complexity](http://wiki.c2.com/?AccidentalComplexity), but most of that accidental complexity is there because it’s so convenient. At every level from the hardware architecture to the syscall interface to the I/O framework we use, we take on complexity, much of which could be eliminated if we could sit down and re-write all of the systems and their interfaces today, but it’s too inconvenient to re-invent the universe to reduce complexity and we get benefits from economies of scale, so we live with what we have.

For those reasons and more, in practice, the solution to poor performance caused by “excess” complexity is often to add more complexity. In particular, the gains we’ve seen that get us back to the quickness of the quickest machines from thirty to forty years ago have come not from listening to exhortations to reduce complexity, but from piling on more complexity.

The `ipad pro` is a feat of modern engineering; the engineering that went into increasing the refresh rate on both the input and the output as well as making sure the software pipeline doesn’t have unnecessary buffering is complex! The design and manufacture of high-refresh-rate displays that can push system latency down is also non-trivially complex in ways that aren’t necessary for bog standard `60 Hz` displays.

This is actually a common theme when working on latency reduction. A common trick to reduce latency is to add a cache, but adding a cache to a system makes it more complex. For systems that generate new data and can’t tolerate a cache, the solutions are often even more complex. An example of this might be [large scale RoCE deployments](https://en.wikipedia.org/wiki/RDMA_over_Converged_Ethernet). These can push remote data access latency from from the millisecond range down to the microsecond range, [which enables new classes of applications](https://danluu.com/infinite-disk/). However, this has come at a large cost in complexity. Early large-scale RoCE deployments easily took tens of person years of effort to get right and also came with a tremendous operational burden.

### Conclusion

It’s a bit absurd that a modern gaming machine running at `4,000x` the speed of an `apple 2`, with a CPU that has `500,000x` as many transistors (with a GPU that has `2,000,000x` as many transistors) can maybe manage the same latency as an `apple 2` in very carefully coded applications if we have a monitor with nearly `3x` the refresh rate. It’s perhaps even more absurd that the default configuration of the `powerspec g405`, which had the fastest single-threaded performance you could get until October 2017, had more latency from keyboard-to-screen (approximately `3 feet`, maybe `10 feet` of actual cabling) than sending a packet around the world (`16187 mi` from NYC to Tokyo to London back to NYC, more due to the cost of running the shortest possible length of fiber).

On the bright side, we’re arguably emerging from the latency dark ages and it’s now possible to assemble a computer or buy a tablet with latency that’s in the same range as you could get off-the-shelf in the 70s and 80s. This reminds me a bit of the screen resolution & density dark ages, where CRTs from the 90s offered better resolution and higher pixel density than affordable non-laptop LCDs until relatively recently. 4k displays have now become normal and affordable 8k displays are on the horizon, blowing past anything we saw on consumer CRTs. I don’t know that we’ll see the same kind improvement with respect to latency, but one can hope. There are individual developers improving the experience for people who use certain, very carefully coded, applications, but it’s not clear what force could cause a significant improvement in the default experience most users see.

### Other posts on latency measurement

- [Terminal latency](https://danluu.com/term-latency/)

- [Keyboard latency](https://danluu.com/keyboard-latency/)

- [Mouse vs. keyboard latency](https://danluu.com/keyboard-v-mouse/) (human factors, not device latency)

- [Editor latency](https://pavelfatin.com/typing-with-pleasure/) (by Pavel Fatin)

- [Windows 10 compositing latency](http://www.lofibucket.com/articles/dwm_latency.html) (by Pekka Vaananen)

- [AR/VR latency](http://blogs.valvesoftware.com/abrash/latency-the-sine-qua-non-of-ar-and-vr/) (by Michael Abrash)

- [Latency mitigation strategies](https://danluu.com/latency-mitigation/) (by John Carmack)

### Appendix: why measure latency?

Latency matters! For very simple tasks, [people can perceive latencies down to `2 ms` or less](https://pdfs.semanticscholar.org/386a/15fd85c162b8e4ebb6023acdce9df2bd43ee.pdf). Moreover, increasing latency is not only noticeable to users, [it causes users to execute simple tasks less accurately](http://www.tactuallabs.com/papers/howMuchFasterIsFastEnoughCHI15.pdf). If you want a visual demonstration of what latency looks like and you don’t have a super-fast old computer lying around, [check out this MSR demo on touchscreen la  tency](https://www.youtube.com/watch?v=vOvQCPLkPt4).

[Throughput](https://stackoverflow.com/a/39187441/334816) also matters, but this is widely understood and measured. If you go to pretty much any mainstream review or benchmarking site, you can find a wide variety of throughput measurements, so there’s less value in writing up additional throughput measurements.

### Appendix: apple 2 keyboard

The `apple 2e`, instead of using a programmed microcontroller to read the keyboard, uses a much simpler custom chip designed for reading keyboard input, the AY 3600. If we look at [the AY 3600 datasheet](https://danluu.com/AY3600.pdf),we can see that the scan time is `(90 * 1/f)` and the debounce time is listed as `strobe_delay`. These quantities are determined by some capacitors and a resistor, which appear to be `47pf`, `100k ohms`, and `0.022uf` for the Apple 2e. Plugging these numbers into [the AY3600 datasheet](https://danluu.com/AY3600.pdf), we can see that `f = 50 kHz`, giving us a `1.8 ms` scan delay and a `6.8 ms` debounce delay (assuming the values are accurate – [capacitors can degrade over time](http://www.kemet.com/Lists/TechnicalArticles/Attachments/191/Why%2047%20uF%20capacitor%20drops%20to%2037%20uF-%2030%20uF-%20or%20lower.pdf), so we should expect the real delays to be shorter on our old Apple 2e), giving us less than `8.6 ms` for the internal keyboard logic.

Comparing to a keyboard with a `167 Hz` scan rate that [scans two extra times to debounce](https://www.arduino.cc/en/Tutorial/Debounce), the equivalent figure is `3 * 6 ms = 18 ms`. With a `100Hz` scan rate, that becomes `3 * 10 ms = 30 ms`. `18 ms` to `30 ms` of keyboard scan plus debounce latency is in line with [what we saw when we did some preliminary keyboard latency measurements](https://danluu.com/keyboard-latency/).

For reference, the ergodox uses a `16 MHz` microcontroller with ~80k transistors and the `apple 2e` CPU is a `1 MHz` chip with 3.5k transistors.

### Appendix: experimental setup

Most measurements were taken with a 240fps camera (`4.167 ms` resolution). Devices with response times below `40 ms` were re-measured with a 1000fps camera (`1 ms` resolution). Results in the tables are the results of multiple runs and are rounded to the nearest `10 ms` to avoid the impression of false precision. For desktop results, results are measured from when the key started moving until the screen finished updating. Note that this is different from most key-to-screen-update measurements you can find online, which typically use a setup that effectively removes much or all of the keyboard latency, which, as an end-to-end measurement, is only realistic if you have a psychic link to your computer (this isn’t to say the measurements aren’t useful – if, as a programmer, you want a reproducible benchmark, it’s nice to reduce measurement noise from sources that are beyond your control, but that’s not relevant to end users). Another significant difference is that measurements were done with settings as close to the default OS settings as possible since approximately 0% of users will futz around with display settings to reduce buffering, disable the compositor, etc.

Computer results were taken using the “default” terminal for the system, [which could easily cause `20 ms` to `30 ms` difference between a fast terminal and a slow terminal](https://danluu.com/term-latency/).

The `powerspec g405` baseline result is using integrated graphics (the machine doesn’t come with a graphics card) and the `60 Hz` result is with a cheap video card.

Mobile results were done by using the default browser, browsing to [https://danluu.com](https://danluu.com/), and measuring the latency from finger movement until the screen first updates to indicate that scrolling has occurred. In the cases where this didn’t make sense, (kindles, gameboy color, etc.), some action that makes sense for the platform was taken (changing pages on the kindle, pressing the joypad on the gameboy color in a game, etc.).

In the case of “ties”, results are ordered by the unrounded latency as a tiebreaker, but this shouldn’t be considered significant. Differences of `10 ms` should probably also not be considered significant.

The `custom haswell-e` was tested with `gsync` on and there was no observable difference. The year for that box is somewhat arbitrary, since the CPU is from `2014`, but the display is newer (I believe you couldn’t get a `165 Hz` display until `2015`.

The number of transistors for some modern machines is a rough estimate because exact numbers aren’t public. Feel free to ping me if you have a better estimate!

The color scales for latency and year are linear and the color scales for clock speed and number of transistors are log scale.

*This is a work in progress. I expect to get benchmarks from a lot more old computers the next time I visit Seattle. If you know of old computers I can test in the NYC area (that have their original displays or something like them), let me know! If you have a device you’d like to donate for testing, feel free to mail it to*

Dan Luu
Recurse Center
455 Broadway, 2nd Floor
New York, NY 10013

*Thanks to [RC](https://www.recurse.com/scout/click?t=b504af89e87b77920c9b60b2a1f6d5e8), David Albert, Bert Muthalaly, Christian Ternus, Kate Murphy, Ikhwan Lee, Peter Bhat Harkins, Leah Hanson, Alicia Thilani Singham Goodwin, Amy Huang, Dan Bentley, Jacquin Mininger, Rob, Raph Levien, Max McCrea, Peter Town, Jon Cinque, Anonymous, and Jonathan Dahan for donating devices to test and thanks to Leah Hanson, Andy Matuschak, Milosz Danczak, @amos, and David Albert for comments/corrections/discussion.*

 [Is there data on the quality of management decisions? →](https://danluu.com/bad-decisions/)

 [Archive](https://danluu.com/)  [Support this site (patreon)](https://www.patreon.com/danluu)  [About](https://danluu.com/about/)  [Twitter](https://twitter.com/danluu)  [RSS](https://danluu.com/atom.xml)