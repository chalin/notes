How We Built r/Place

![place-final.png](../_resources/609fec766e37a22c1c9a29ac95850108.png)

# How We Built r/Place

[Technology](https://redditblog.com/topic/technology/)  [Staff](https://redditblog.com/author/blabyrinth/) • [April 13, 2017](https://redditblog.com/2017/04/13/how-we-built-rplace/)

**Brian Simpson, Matt Lee, & Daniel Ellis****
***(u/bsimpson, u/madlee, & u/daniel)***
**

**Each year for April Fools’**, rather than a prank, we like to create a project that explores the way that humans interact at large scales. This year we came up with Place, a collaborative canvas on which a single user could only place a single tile every five minutes. This limitation de-emphasized the importance of the individual and necessitated the collaboration of many users in order to achieve complex creations. Each tile placed was relayed to observers in real-time.

Multiple engineering teams (frontend, backend, mobile) worked on the project and most of it was built using existing technology at Reddit. This post details how we approached building Place from a technical perspective.

But first, if you want to check out the code for yourself, you can find it [here](http://github.com/reddit/reddit-plugin-place-opensource). And if you’re interested in working on projects like Place in the future, [we’re hiring](https://about.reddit.com/careers/)!

## **Requirements**

Defining requirements for an April Fools’ project is extremely important because it will launch with zero ramp-up and be available immediately to all of Reddit’s users. If it doesn’t work perfectly out of the gate, it’s unlikely to attract enough users to make for an interesting experience.

- The board must be 1000 tiles by 1000 tiles so it feels very large.
- All clients must be kept in sync with the same view of the current board state, otherwise users with different versions of the board will have difficulty collaborating.
- We should support at least 100,000 simultaneous users.
- Users can place one tile every 5 minutes, so we must support an average update rate of 100,000 tiles per 5 minutes (333 updates/s).
- The project must be designed in such a way that it’s unlikely to affect the rest of the site’s normal function even with very high traffic to r/place.
- The configuration must be flexible in case there are unexpected bottlenecks or failures. This means that board size and tile cooldown should be adjustable on the fly in case data sizes are too large or update rates are too high.
- The API should be generally open and transparent so the reddit community can build on it (bots, extensions, data collection, external visualizations, etc) if they choose to do so.

## **Backend**

### **Implementation decisions**

The main challenge for the backend was keeping all the clients in sync with the state of the board. Our solution was to initialize the client state by having it listen for real-time tile placements immediately and then make a request for the full board. The full board in the response could be a few seconds stale as long as we also had real-time placements starting from before it was generated. When the client received the full board it replayed all the real-time placements it received while waiting. All subsequent tile placements could be drawn to the board immediately as they were received.

For this scheme to work we needed the request for the full state of the board to be as fast as possible. Our initial approach was to store the full board in a [single row in Cassandra](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/understand_the_cassandra_data_model.html) and each request for the full board would read that entire row. The format for each column in the row was:

(x, y): {‘timestamp’: epochms, ‘author’: user_name, ‘color’: color}

Because the board contained 1 million tiles this meant that we had to read a row with 1 million columns. On our production cluster this read took up to 30 seconds, which was unacceptably slow and could have put excessive strain on Cassandra.

Our next approach was to store the full board in redis. We used a [bitfield](https://redis.io/commands/bitfield) of 1 million 4 bit integers. Each 4 bit integer was able to encode a 4 bit color, and the x,y coordinates were determined by the offset (offset = x + 1000y) within the bitfield. We could read the entire board state by reading the entire bitfield. We were able to update individual tiles by updating the value of the bitfield at a specific offset (no need for locking or read/modify/write). We still needed to store the full details in Cassandra so that users could inspect individual tiles to see who placed them and when. We also planned on using Cassandra to restore the board in case of a redis failure. Reading the entire board from redis took less than 100ms, which was fast enough.

*Illustration showing how colors were stored in redis, using a 2×2 board:*
![drawio-1.png](../_resources/df6e8d38c4a15e99ac167f716c263cd2.png)

We were concerned about exceeding maximum read bandwidth on redis. If many clients connected or refreshed at once they would simultaneously request the full state of the board, all triggering reads from redis. Because the board was a shared global state the obvious solution was to use caching. We decided to cache at the CDN (Fastly) layer because it was simple to implement and it meant the cache was as close to clients as possible which would help response speed. Requests for the full state of the board were cached by Fastly with an expiration of 1 second. We also added the [stale-while-revalidate](https://docs.fastly.com/guides/performance-tuning/serving-stale-content#usage) cache control header option to prevent more requests from falling through than we wanted when the cached board expired. [Fastly maintains around 33 POPs](https://www.fastly.com/network-map) which do independent caching, so we expected to get at most 33 requests per second for the full board.

We used our [websocket service](https://github.com/reddit/reddit-service-websockets) to publish updates to all the clients. We’ve had success using it in production for [reddit live](https://www.reddit.com/live) threads with over 100,000 simultaneous viewers, live PM notifications, and other features. The websocket service has also been a cornerstone of our past April Fools projects such as The Button and Robin. For r/place, clients maintained a websocket connection to receive real-time tile placement updates.

### **API**

#### **Retrieve the full board**

![board-bitmap.png](../_resources/800a9ef366ff49bf407e488ff07fba2d.png)

Requests first went to Fastly. If there was an unexpired copy of the board it would be returned immediately without hitting the reddit application servers. Otherwise, if there was a cache miss or the copy was too old, the reddit application would read the full board from redis and return that to Fastly to be cached and returned to the client.

*Request rate and response time as measured by the reddit application:*

Notice that the request rate never exceeds 33/s, meaning that the caching by Fastly was very effective at preventing most requests from hitting the reddit application.

When a request did hit the reddit application the read from redis was very fast.

#### **Draw a tile**

![draw.png](../_resources/6c79fa032cb3e606700a91c4571a1f4d.png)
The steps for drawing a tile were:

1. Read the timestamp of the user’s last tile placement from Cassandra. If it was more recent than the cooldown period (5 minutes) reject the draw attempt and return an error to the user.

2. Write the tile details to redis and Cassandra.
3. Write the current timestamp as the user’s last tile placement in Cassandra.

4. Tell the websocket service to send a message to all connected clients with the new tile.

All reads and writes to Cassandra were done with [consistency level QUORUM](http://docs.datastax.com/en/archived/cassandra/1.2/cassandra/dml/dml_config_consistency_c.html) to ensure strong consistency.

We actually had a race condition here that allowed users to place multiple tiles at once. There was no locking around the steps 1-3 so simultaneous tile draw attempts could all pass the check at step 1 and then draw multiple tiles at step 2. It seems that some users discovered this error or had bots that didn’t gracefully follow the ratelimits so there were about 15,000 tiles drawn that abused this error (~0.09% of all tiles placed).

*Request rate and response time as measured by the reddit application:*

We experienced a maximum tile placement rate of almost 200/s. This was below our calculated maximum rate of 333/s (average of 100,000 users placing a tile every 5 minutes).

#### **Get details of a single tile**

![pixel1.png](../_resources/55fcf6f67c05af6f7cb1067ab8920b1c.png)
Requests for individual tiles resulted in a read straight from Cassandra.
*Request rate and response time as measured by the reddit application:*

This endpoint was very popular. In addition to regular client requests, people wrote scrapers to retrieve the entire board one tile at a time. Since this endpoint wasn’t cached by the CDN, all requests ended up being served by the reddit application.

Response times for these requests were pretty fast and stable throughout the project.

### **Websockets**

We don’t have isolated metrics for r/place’s effect on the websocket service, but we can estimate and subtract the baseline use from the values before the project started and after it ended.

*Total connections to the websocket service:*

The baseline before r/place began was around 20,000 connections and it peaked at 100,000 connections, so we probably had around 80,000 users connected to r/place at its peak.

*Websocket service bandwidth:*

At the peak of r/place the websocket service was transmitting over 4 gbps (150 Mbps per instance and 24 instances).

## **Frontend: Web and Mobile Clients**

Building the frontend for Place involved many of the challenges for cross-platform app development. We wanted Place to be a seamless experience on all of our major platforms including desktop web, mobile web, iOS and Android.

The UI in place needed to do three important things:
1. Display the state of the board in real time
2. Facilitate user interaction with the board
3. Work on all of our platforms, including our mobile apps

The main focus of the UI was the canvas, and the [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) was a perfect fit for it. We used a single 1000 x 1000 <canvas> element, drawing each tile as a single pixel.

### **Drawing the canvas**

The canvas needed to represent the state of the board in real time. We needed to draw the state of the entire board when the page loaded, and draw updates to the board state that came in over websockets. There are generally three ways to go about updating a canvas element using the [CanvasRenderingContext2D](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) interface:

1. Drawing an existing image onto the canvas using [drawImage()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage)

2. Draw shapes with the various shape drawing methods, e.g. using [fillRect()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/fillRect) to fill a rectangle with a color

3. Construct an ImageData object and paint it into the canvas using [putImageData()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/putImageData)

The first option wouldn’t work for us since since we didn’t already have the board in image form, leaving options 2 and 3. Updating individual tiles using fillRect() was very straightforward: when a websocket update comes in, just draw a 1 x 1 rectangle at the (x, y) position. This worked OK in general, but wasn’t great for drawing the initial state of the board. The putImageData() method was a much better fit for this, since we were able to define the color of each pixel in a single ImageData object and draw the whole canvas at once.

#### **Drawing the initial state of the board**

Using putImageData() requires defining the board state as a [Uint8ClampedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray), where each value is an 8-bit unsigned integer clamped to 0-255. Each value represents a single color channel (red, green, blue, and alpha), and each pixel requires 4 items in the array. A 2 x 2 canvas would require a 16-byte array, with the first 4 bytes representing the top left pixel on the canvas, and the last 4 bytes representing the bottom right pixel.

*Illustration showing how canvas pixels relate to their Uint8ClampedArray representation:*

![drawio-2.png](../_resources/81e17af7cc93b820e2e337c3cc69a721.png)
For place’s canvas, the array is 4 million bytes long, or 4MB.

On the backend, the board state is stored as a 4-bit bitfield. Each color is represented by a number between 0 and 15, allowing us to pack 2 pixels of color information into each byte. In order to use this on the client, we needed to do 3 things:

1. Pull the binary data down to the client from our API
2. “Unpack” the data
3. Map the 4-bit colors to useable 32-bit colors

To pull down the binary data, we used the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) in browsers that support it. For those that don’t, we fell back to a normal [XMLHttpRequest](https://www.youtube.com/watch?v=Pubd-spHN-0) with responseType set to “arraybuffer”.

The binary data we receive from the API contains 2 pixels of color data in each byte. The smallest [TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) constructors we have allow us to work with binary data in 1-byte units. This is inconvenient for use on the client so the first thing we do is to “unpack” that data so it’s easier to work with. This process is straightforward, we just iterate over the packed data and split out the high and low order bits, copying them into separate bytes of another array. Finally, the 4-bit color values needed to be mapped to useable 32-bit colors.

|     |     |     |
| --- | --- | --- |
| API Response | 0x47 | 0xE9 |
| Unpacked | 0x04 | 0x07 | 0x0E | 0x09 |
| Mapped to 32bit colors | 0xFFA7D1FF | 0xA06A42FF | 0xCF6EE4FF | 0x94E044FF |

The ImageData structure needed to use the putImageData() method requires the end result to be readable as a Uint8ClampedArray with the color channel bytes in RGBA order. This meant we needed to do another round of “unpacking”, splitting each color into its component channel bytes and putting them into the correct index. Needing to do 4 writes per pixel was also inconvenient, but luckily there was another option.

TypedArray objects are essentially array *views* into ArrayBuffer instances, which actually represent the binary data. One neat thing about them is that multiple TypedArray instances can read and write to the same underlying ArrayBuffer instance. Instead of writing 4 values into an 8-bit array, we could write a single value into a 32-bit array!  Using a Uint32Array to write, we were able to easily update a tile’s color by updating a single array index. The only change required was that we had to store our color palette in reverse-byte order (ABGR) so that the bytes automatically fell in the correct position when read using the Uint8ClampedArray.

|     |     |     |     |
| --- | --- | --- | --- |
| 0   | 1   | 2   | 3   |
| 0xFFD1A7FF | 0xFF426AA0 | 0xFFE46ECF | 0xFF44E094 |
| 255 | 167 | 209 | 255 | 160 | 106 | 66  | 255 | 207 | 110 | 228 | 255 | 148 | 224 | 68  | 255 |
| *r* | *g* | *b* | *a* | *r* | *g* | *b* | *a* | *r* | *g* | *b* | *a* | *r* | *g* | *b* | *a* |

#### **Handling websocket updates**

Using the drawRect() method was working OK for drawing individual pixel updates as they came in, but it had one major drawbacks: large bursts of updates coming in at the same time could cripple browser performance. We knew that updates to the board state would be very frequent, so we needed to address this issue.

Instead of redrawing the canvas immediately each time a websocket update came in, we wanted to be able to batch multiple websocket updates that come in around the same time and draw them all at once. We made two changes to do this:

1. We stopped using drawRect() altogether, since we’d already figured out a nice convenient way of updating many pixels at once with putImageData()

2. We moved the actual canvas drawing into a requestAnimationFrame loop

By moving the drawing into an animation loop, we were able to write websocket updates to the ArrayBuffer immediately and defer the actual drawing. All websocket updates in between frames (about 16ms) were batched into a single draw. Because we used requestAnimationFrame, this also meant that if draws took too long (longer than 16ms), only the refresh rate of the canvas would be affected (rather than crippling the entire browser).

### **Interacting with the Canvas**

Equally importantly, the canvas needed to facilitate user interaction. The core way that users can interact with the canvas is to place tiles on it. Precisely drawing individual pixels at 100% scale would be extremely painful and error prone, so we also needed to be able to zoom in (a lot!). We also needed to be able to pan around the canvas easily, since it was too large to fit on most screens (especially when zoomed in).

#### **Camera zoom**

Users were only allowed to draw tiles once every 5 minutes, so misplaced tiles would be especially painful. We had to zoom in on the canvas enough that each tile would be a fairly large target for drawing. This was especially important for touch devices. We used a 40x scale for this, giving each tile a 40 x 40 target area. To apply the zoom, we wrapped the <canvas> element in a <div> that we applied a CSS transform: scale(40, 40) to. This worked great for placing tiles, but wasn’t ideal for *viewing* the board (especially on small screens), so we made this toggleable between two zoom levels: 40x for drawing, 4x for viewing.

Using CSS to scale up the canvas made it easy to keep the code that handled drawing the board separate from the code that handled scaling, but unfortunately this approach had some issues. When scaling up an image (or canvas), browsers default to algorithms that apply “smoothing” to the image. This works OK in some cases, but it *completely* ruins pixel art by turning it into a blurry mess. The good news it that there’s another CSS, [image-rendering](https://developer.mozilla.org/en-US/docs/Web/CSS/image-rendering),  which allows us to ask browsers to not do that. The bad news is that not all browsers fully support that property.

*Bad news blurs:**
*
*![www-reddit-com-place-webviewtrue-1.png](../_resources/c156df2e1a9455c7d85fb151503406e2.png)*

We needed another way to scale up the canvas for these browsers. I mentioned earlier on that there are generally three ways to go about drawing to a canvas. The first method, drawImage(), supports drawing an existing image *or another canvas* into a canvas. It also supports scaling that image up or down when drawing it, and though upscaling has the same blurring issue by default that upscaling in CSS has, this can be disabled in a more cross-browser compatible way by turning off the [CanvasRenderingContext2D.imageSmoothingEnabled](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/imageSmoothingEnabled) flag.

So the fix for our blurry canvas problem was to add another step to the rendering process. We introduced another <canvas> element, this one sized and positioned to fit across the container element (i.e. the viewable area of the board). After redrawing the canvas, we use drawImage() to draw the visible portion of it into this new display canvas at the proper scale. Since this extra step adds a little overhead to the rendering process, we only did this for browsers that don’t support the CSS image-rendering property.

#### **Camera pan**

The canvas is a fairly big image, especially when zoomed in, so we needed to provide ways of navigating it. To adjust the position of the canvas on the screen, we took a similar approach to what we did with scaling: we wrapped the <canvas> element in another <div> that we applied CSS transform: translate(x, y) to. Using a separate div made it easy to control the order that these transforms were applied to the canvas, which was important for preventing the camera from moving when toggling the zoom level.

We ended up supporting a variety of ways to adjust the camera position, including:

- Click and drag
- Click to move
- Keyboard navigation

Each of these methods required a slightly different approach.

##### Click-and-drag

The primary way of navigating was click-and-drag (or touch-and-drag). We stored the x, y position of the mousedown event. On each mousemove event, we found the offset of the mouse position relative to that start position, then added that offset to the existing saved canvas offset. The camera position was updated immediately so that this form of navigation felt really responsive.

##### Click-to-move

We also allowed clicking on a tile to center that tile on the screen. To accomplish this, we had to keep track of the distance moved between the mousedown and mouseup events, in order to distinguish “clicks” from “drags”. If the mouse did not move enough to be considered a “drag”, we adjusted the camera position by the difference between the mouse position and the point at the center of the screen. Unlike click-and-drag movement, the camera position was updated with an easing function applied. Instead of setting the new position immediately, we saved it as a “target” position. Inside the animation loop (the same one used to redraw the canvas), we moved the current camera position closer to the target using an easing function. This prevented the camera move from feeling too jarring.

##### Keyboard navigation

We also supported navigating with the keyboard, using either the WASD keys or the arrow keys. The four direction keys controlled an internal **movement vector**. This vector defaulted to (0, 0) when no movement keys were down, and each of the direction keys added or subtracted 1 from either the x or y component of the vector when pressed. For example, pressing the “right” and “up” keys would set the movement vector to (1, -1). This movement vector was then used inside the animation loop to move the camera.

During the animation loop, a **movement speed** was calculated based on the current zoom level using the formula:

movementSpeed = maxZoom / currentZoom * speedMultiplier

This made keyboard navigation faster when zoomed out, which felt a lot more natural.

The movement vector is then [normalized](http://mathworld.wolfram.com/NormalizedVector.html) and multiplied by the movement speed, then applied to the current camera position. We normalized the vector to make sure diagonal movement was the same speed as orthogonal movement, which also helped it feel more natural. Finally, we applied the same kind of easing function to changes to the movement vector itself. This smoothed out changes in movement direction and speed, making the camera feel much more fluid and [juicy](https://www.youtube.com/watch?v=Fy0aCDmgnxg).

### **Mobile app support**

There were a couple of additional challenges to embedding the canvas in the mobile apps for iOS and Android. First, we needed to authenticate the user so they could place tiles. Unlike on the web, where authentication is session based, with the mobile apps we use OAuth. This means that the app needs to provide the webview with an access token for the currently logged in user. The safest way to do this was to inject the oauth authorization headers by making a javascript call from the app to the webview (this would’ve also allowed us to set other headers if needed). It was then a matter of passing the authorization headers along with each api call.

r.place.injectHeaders({‘Authorization’: ‘Bearer <access token>’});

For the iOS side we additionally implemented notification support when your next tile was ready to be placed on the canvas. Since tile placement occurred completely in the webview we needed to implement a callback to the native app. Fortunately with iOS 8 and higher this is possible with a simple javascript call:

webkit.messageHandlers.tilePlacedHandler.postMessage(this.cooldown / 1000);

The delegate method in the app then schedules a notification based on the cooldown timer that was passed in.

![place-notif.png](../_resources/e959576922b2387774c577e847663b26.png)

## **What We Learned**

### **You’ll always miss something**

Since we had planned everything out perfectly, we knew when we launched, nothing could possibly go wrong. We had load tested the frontend, load tested the backend, there was simply no way we humans could have made any other mistakes.

Right?

The launch went smoothly. Over the course of the morning, as the popularity of r/place went up, so did the number of connections and traffic to our websockets instances:

No big deal, and exactly what we expected. Strangely enough, we thought we were network-bound on those instances and figured we had a lot more headway. Looking at the CPU of the instances, however, painted a different picture:

Those are 8-core instances, so it was clear they were reaching their limits. Why were these boxes suddenly behaving so differently? We chalked it up to place being a much different workload type than they’d seen before. After all, these were lots of very tiny messages; we typically send out larger messages like live thread updates and notifications. We also usually don’t have *that* many people all receiving the same message, so a lot of things were different.

Still, no big deal, we figured we’d just scale it and call it a day. The on-call person doubled the number of instances and went to a doctor’s appointment, not a care in the world.

Then, this happened:

That graph may seem unassuming if it weren’t for the fact that it was for our production Rabbit MQ instance, which handles not only our websockets messages but basically everything that reddit.com relies on. And it wasn’t happy; it wasn’t happy at all.

After a lot of investigating, hand-wringing, and instance upgrading, we narrowed down the problem to the management interface. It had always seemed kind of slow, and we realized that the [rabbit diamond collector](https://github.com/python-diamond/Diamond/blob/master/src/collectors/rabbitmq/rabbitmq.py) we use for getting our stats was querying it regularly. We believe that the additional exchanges created when launching new websockets instances, combined with the throughput of messages we were receiving on those exchanges, caused rabbit to buckle while trying to do bookkeeping to do queries for the admin interface. So we turned it off, and things got better.

We don’t like being in the dark, so we whipped up an artisanal, hand-crafted monitoring script to get us through the project:

`$ cat s****y_diamond.sh`#!/bin/bash/usr/sbin/rabbitmqctl list_queues | /usr/bin/awk '$2~/[0-9]/{print "servers.foo.bar.rabbit.rabbitmq.queues." $1 ".messages " $2 " " systime()}' | /bin/grep -v 'amq.gen' | /bin/nc 10.1.2.3 2013

If you’re wondering why we kept adjusting the timeouts on placing pixels, there you have it. We were trying to relieve pressure to keep the whole project running. This is also the reason why, during one period, some pixels were taking a long time to show up.

So unfortunately, despite what messages like this would have you believe:

The reasons for the adjustments were entirely technical. Although it was cool to watch r/place/new after making the change:

![place-page.png](../_resources/475ed2cce09a3c39b6db9572b4a034fd.png)
So maybe *that* was part of the motivation.

### **Bots Will Be Bots**

We ran into one more slight hiccup at the end of the project. In general, one of our recurring problems is clients with bad retry behavior. A lot of clients, when faced with an error, will simply retry. And retry. And retry. This means whenever there is a hiccup on the site, it can often turn into a retry storm from some clients who have not been programmed to [back-off](https://en.wikipedia.org/wiki/Exponential_backoff) in the case of trouble.

When we turned off place, the endpoints that a lot of bots were hitting started returning non-200s. Code like [this](https://github.com/PlaceStart/placestart/blob/7d27beacc75dcf84d634ea32a5bf0421bf2c042a/monitor.py#L114-L125) wasn’t very nice. Thankfully, this was easy to block at the Fastly layer.

## **Creating Something More**

This project could not have come together so successfully without a tremendous amount of teamwork. We’d like to thank u/gooeyblob, u/egonkasper, u/eggplanticarus, u/spladug, u/thephilthe, u/d3fect and everyone else who contributed to the r/place team, for making this April Fools’ experiment possible.

And as we mentioned before, if you’re interested in creating unique experiences for millions of users, check out [our Careers page](https://about.reddit.com/careers/).

* * *

*Want to discuss this blog post? Join the r/place team in the comments on [r/programming](http://reddit.com/r/programming).*

- [Share on Facebook (Opens in new window)](https://redditblog.com/2017/04/13/how-we-built-rplace/?share=facebook&nb=1)
- [Click to share on Twitter (Opens in new window)](https://redditblog.com/2017/04/13/how-we-built-rplace/?share=twitter&nb=1)
- [Click to share on Pinterest (Opens in new window)](https://redditblog.com/2017/04/13/how-we-built-rplace/?share=pinterest&nb=1)
- [Click to share on Tumblr (Opens in new window)](https://redditblog.com/2017/04/13/how-we-built-rplace/?share=tumblr&nb=1)
- [Click to share on Reddit (Opens in new window)](https://redditblog.com/2017/04/13/how-we-built-rplace/?share=reddit&nb=1)

-
.
.
.