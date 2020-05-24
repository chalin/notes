Micro Frontends - extending the microservice idea to frontend development

 [ # Micro Frontends](http://micro-frontends.org/)

## extending the microservice idea to frontend development

 [View project on GitHub](https://github.com/neuland/micro-frontends)

Techniques, strategies and recipes for building a **modern web app** with **multiple teams** using **different JavaScript frameworks**.

## //What are Micro Frontends?

The term **Micro Frontends** first came up in [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar/techniques/micro-frontends) at the end of 2016. It extends the concepts of micro services to the frontend world. The current trend is to build a feature-rich and powerful browser application, aka single page app, which sits on top of a micro service architecture. Over time the frontend layer, often developed by a separate team, grows and gets more difficult to maintain. That’s what we call a [Frontend Monolith](https://www.youtube.com/watch?v=pU1gXA0rfwc).

The idea behind Micro Frontends is to think about a website or web app as **a composition of features** which are owned by **independent teams**. Each team has a **distinct area of business** or **mission** it cares about and specialises in. A team is **cross functional** and develops its features **end-to-end**, from database to user interface.

However, this idea is not new, in the past it went by the name of [Frontend Integration for Verticalised Systems](https://dev.otto.de/2014/07/29/scaling-with-microservices-and-vertical-decomposition/) or [Self contained Systems](http://scs-architecture.org/). But Micro Frontends is clearly a more friendly and less bulky term.

**Monolithic Frontends**![](../_resources/a8dab3c1f14d76c8552e9bdfeb03884d.png)
**Organisation in Verticals**![](../_resources/c8dc031a4cca43c4ff7ecb059bb69771.png)

## //What’s a Modern Web App?

In the introduction I’ve used the phrase “building a modern web app”. Let’s define the assumptions that are connected with this term.

To put this into a broader perspective, [Aral Balkan](https://ar.al/) has written a blog post about what he calls the [Documents‐to‐Applications Continuum](https://ar.al/notes/the-documents-to-applications-continuum/). He comes up with the concept of a sliding scale where a site, built out of **static documents**, connected via links, is **on the left** end and a pure behaviour driven, **contentless application** like an online photo editor is **on the right**.

If you would position your project on the **left side of this spectrum**, an **integration on webserver level** is a good fit. With this model a server collects and **concatenates HTML strings** from all components that make up the page requested by the user. Updates are done by reloading the page from the server or replacing parts of it via ajax. [Gustaf Nilsson Kotte](https://twitter.com/gustaf_nk/) has written a [comprehensive article](https://gustafnk.github.io/microservice-websites/) on this topic.

When your user interface has to provide **instant feedback**, even on unreliable connections, a pure server rendered site is not sufficient anymore. To implement techniques like [Optimistic UI](https://www.smashingmagazine.com/2016/11/true-lies-of-optimistic-user-interfaces/) or [Skeleton Screens](http://www.lukew.com/ff/entry.asp?1797) you need to be able to also **update** your UI **on the device itself**. Google’s term [Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/) aptly describes the **balancing act** of being a good citizen of the web (progressive enhancement) while also providing app-like performance. This kind of application is located somewhere **around the middle of the site-app-continuum**. Here a solely server based solution is not sufficient anymore. We have to move the **integration into the browser**, and this is the focus of this article.

## //Core Ideas behind Micro Frontends

- **Be Technology Agnostic**

Each team should be able to choose and upgrade their stack without having to coordinate with other teams. [Custom Elements](https://micro-frontends.org/#the-dom-is-the-api) are a great way to hide implementation details while providing a neutral interface to others.

- **Isolate Team Code**

Don’t share a runtime, even if all teams use the same framework. Build independent apps that are self contained. Don’t rely on shared state or global variables.

- **Establish Team Prefixes**

Agree on naming conventions where isolation is not possible yet. Namespace CSS, Events, Local Storage and Cookies to avoid collisions and clarify ownership.

- **Favor Native Browser Features over Custom APIs**

Use [Browser Events for communication](https://micro-frontends.org/#parent-child-communication--dom-modification) instead of building a global PubSub system. If you really have to build a cross team API, try keeping it as simple as possible.

- **Build a Resilient Site**

Your feature should be useful, even if JavaScript failed or hasn’t executed yet. Use [Universal Rendering](https://micro-frontends.org/#serverside-rendering--universal-rendering) and Progressive Enhancement to improve perceived performance.

* * *

## //The DOM is the API

[Custom Elements](https://developers.google.com/web/fundamentals/getting-started/primers/customelements), the interoperability aspect from the Web Components Spec, are a good primitive for integration in the browser. Each team builds their component **using their web technology of choice** and **wraps it inside a Custom Element** (e.g. `<order-minicart></order-minicart>`). The DOM specification of this particular element (tag-name, attributes & events) acts as the contract or public API for other teams. The advantage is that they can use the component and its functionality without having to know the implementation. They just have to be able to interact with the DOM.

But Custom Elements alone are not the solution to all our needs. To address progressive enhancement, universal rendering or routing we need additional pieces of software.

This page is divided into two main areas. First we will discuss [Page Composition](https://micro-frontends.org/#page-composition) - how to assemble a page out of components owned by different teams. After that we’ll show examples for implementing clientside [Page Transition](https://micro-frontends.org/#page-transition).

## //Page Composition

Beside the **client-** and **serverside** integration of code written in **different frameworks** itself, there are a lot of side topics that should be discussed: mechanisms to **isolate js**, **avoid css conflicts**, **load resources** as needed, **share common resources** between teams, handle **data fetching** and think about good **loading states** for the user. We’ll go into these topics one step at a time.

### ///The Base Prototype

The product page of this model tractor store will serve as the basis for the following examples.

It features a **variant selector** to switch between the three different tractor models. On change product image, name, price and recommendations are updated. There is also a **buy button**, which adds the selected variant to the basket and a **mini basket** at the top that updates accordingly.

[![model-store-0.gif](../_resources/a7848890dd78e95a22ce57f44cf81ec3.gif)](https://micro-frontends.org/0-model-store/)

[try in browser](https://micro-frontends.org/0-model-store/) & [inspect the code](https://github.com/neuland/micro-frontends/tree/master/0-model-store)

All HTML is generated client side using **plain JavaScript** and ES6 Template Strings with **no dependencies**. The code uses a simple state/markup separation and re-renders the entire HTML client side on every change - no fancy DOM diffing and **no universal rendering** for now. Also **no team separation** - [the code](https://github.com/neuland/micro-frontends/tree/master/0-model-store) is written in one js/css file.

### ///Clientside Integration

In this example, the page is split into separate components/fragments owned by three teams. **Team Checkout** (blue) is now responsible for everything regarding the purchasing process - namely the **buy button** and **mini basket**. **Team Inspire** (green) manages the **product recommendations** on this page. The page itself is owned by **Team Product** (red).

[![](../_resources/3d6c88d1a676ef8b85e4b675f28d8cc9.png)](https://micro-frontends.org/1-composition-client-only/)

[try in browser](https://micro-frontends.org/1-composition-client-only/) & [inspect the code](https://github.com/neuland/micro-frontends/tree/master/1-composition-client-only)

**Team Product** decides which functionality is included and where it is positioned in the layout. The page contains information that can be provided by Team Product itself, like the product name, image and the available variants. But it also includes fragments (Custom Elements) from the other teams.

### ///How to Create a Custom Element?

Lets take the **buy button** as an example. Team Product includes the button simply adding `<blue-buy sku="t_porsche"></blue-buy>` to the desired position in the markup. For this to work, Team Checkout has to register the element `blue-buy` on the page.

	class BlueBuy extends HTMLElement {
	  constructor() {
	    super();
	    this.innerHTML = `<button type="button">buy for 66,00 €</button>`;
	  }
	  disconnectedCallback() { ... }
	}
	window.customElements.define('blue-buy', BlueBuy);

Now every time the browser comes across a new `blue-buy` tag, the constructor is called. `this` is the reference to the root DOM node of the custom element. All properties and methods of a standard DOM element like `innerHTML` or `getAttribute()` can be used.

![custom-element.gif](../_resources/29486a05e240a5ebc383b2ce8b4b104a.gif)

When naming your element the only requirement the spec defines is that the name must **include a dash (-)** to maintain compatibility with upcoming new HTML tags. In the upcoming examples the naming convention `[team_color]-[feature]` is used. The team namespace guards against collisions and this way the ownership of a feature becomes obvious, simply by looking at the DOM.

### ///Parent-Child Communication / DOM Modification

When the user selects another tractor in the **variant selector**, the **buy button has to be updated** accordingly. To achieve this Team Product can simply **remove** the existing element from the DOM **and insert** a new one.

	container.innerHTML;
	// => <blue-buy sku="t_porsche">...</blue-buy>
	container.innerHTML = '<blue-buy sku="t_fendt"></blue-buy>';

The `disconnectedCallback` of the old element gets invoked synchronously to provide the element with the chance to clean up things like event listeners. After that the `constructor` of the newly created `t_fendt` element is called.

Another more performant option is to just update the `sku` attribute on the existing element.

	document.querySelector('blue-buy').setAttribute('sku', 't_fendt');

If Team Product used a templating engine that features DOM diffing, like React, this would be done by the algorithm automatically.

![custom-element-attribute.gif](../_resources/314c201162be68d16801036a65efc6a5.gif)

To support this the Custom Element can implement the `attributeChangedCallback` and specify a list of `observedAttributes` for which this callback should be triggered.

	const prices = {
	  t_porsche: '66,00 €',
	  t_fendt: '54,00 €',
	  t_eicher: '58,00 €',
	};

	class BlueBuy extends HTMLElement {
	  static get observedAttributes() {
	    return ['sku'];
	  }
	  constructor() {
	    super();
	    this.render();
	  }
	  render() {
	    const sku = this.getAttribute('sku');
	    const price = prices[sku];
	    this.innerHTML = `<button type="button">buy for ${price}</button>`;
	  }
	  attributeChangedCallback(attr, oldValue, newValue) {
	    this.render();
	  }
	  disconnectedCallback() {...}
	}
	window.customElements.define('blue-buy', BlueBuy);

To avoid duplication a `render()` method is introduced which is called from `constructor` and `attributeChangedCallback`. This method collects needed data and innerHTML’s the new markup. When deciding to go with a more sophisticated templating engine or framework inside the Custom Element, this is the place where its initialisation code would go.

### ///Browser Support

The above example uses the Custom Element V1 Spec which is currently [supported in Chrome, Safari and Opera](http://caniuse.com/#feat=custom-elementsv1). But with [document-register-element](https://github.com/WebReflection/document-register-element) a lightweight and battle-tested polyfill is available to make this work in all browsers. Under the hood, it uses the [widely supported](http://caniuse.com/#feat=mutationobserver) Mutation Observer API, so there is no hacky DOM tree watching going on in the background.

### ///Framework Compatibility

Because Custom Elements are a web standard, all major JavaScript frameworks like Angular, React, Preact, Vue or Hyperapp support them. But when you get into the details, there are still a few implementation problems in some frameworks. At [Custom Elements Everywhere](https://custom-elements-everywhere.com/)  [Rob Dodson](https://twitter.com/rob_dodson) has put together a compatibility test suite that highlights unresolved issues.

### ///Child-Parent or Siblings Communication / DOM Events

But passing down attributes is not sufficient for all interactions. In our example the **mini basket should refresh** when the user performs a **click on the buy button**.

Both fragments are owned by Team Checkout (blue), so they could build some kind of internal JavaScript API that lets the mini basket know when the button was pressed. But this would require the component instances to know each other and would also be an isolation violation.

A cleaner way is to use a PubSub mechanism, where a component can publish a message and other components can subscribe to specific topics. Luckily browsers have this feature built-in. This is exactly how browser events like `click`, `select` or `mouseover` work. In addition to native events there is also the possibility to create higher level events with `new CustomEvent(...)`. Events are always tied to the DOM node they were created/dispatched on. Most native events also feature bubbling. This makes it possible to listen for all events on a specific sub-tree of the DOM. If you want to listen to all events on the page, attach the event listener to the window element. Here is how the creation of the `blue:basket:changed`-event looks in the example:

	class BlueBuy extends HTMLElement {
	  [...]
	  connectedCallback() {
	    [...]
	    this.render();
	    this.firstChild.addEventListener('click', this.addToCart);
	  }
	  addToCart() {
	    // maybe talk to an api
	    this.dispatchEvent(new CustomEvent('blue:basket:changed', {
	      bubbles: true,
	    }));
	  }
	  render() {
	    this.innerHTML = `<button type="button">buy</button>`;
	  }
	  disconnectedCallback() {
	    this.firstChild.removeEventListener('click', this.addToCart);
	  }
	}

The mini basket can now subscribe to this event on `window` and get notified when it should refresh its data.

	class BlueBasket extends HTMLElement {
	  connectedCallback() {
	    [...]
	    window.addEventListener('blue:basket:changed', this.refresh);
	  }
	  refresh() {
	    // fetch new data and render it
	  }
	  disconnectedCallback() {
	    window.removeEventListener('blue:basket:changed', this.refresh);
	  }
	}

With this approach the mini basket fragment adds a listener to a DOM element which is outside its scope (`window`). This should be ok for many applications, but if you are uncomfortable with this you could also implement an approach where the page itself (Team Product) listens to the event and notifies the mini basket by calling `refresh()` on the DOM element.

	// page.js
	const $ = document.getElementsByTagName;

	$('blue-buy')[0].addEventListener('blue:basket:changed', function() {
	  $('blue-basket')[0].refresh();
	});

Imperatively calling DOM methods is quite uncommon, but can be found in [video element api](https://developer.mozilla.org/de/docs/Web/HTML/Using_HTML5_audio_and_video#Controlling_media_playback) for example. If possible the use of the declarative approach (attribute change) should be preferred.

## //Serverside Rendering / Universal Rendering

Custom Elements are great for integrating components inside the browser. But when building a site that is accessible on the web, chances are that initial load performance matters and users will see a white screen until all js frameworks are downloaded and executed. Additionally, it’s good to think about what happens to the site if the JavaScript fails or is blocked. [Jeremy Keith](https://adactio.com/) explains the importance in his ebook/podcast [Resilient Web Design](https://resilientwebdesign.com/). Therefore the ability to render the core content on the server is key. Sadly the web component spec does not talk about server rendering at all. No JavaScript, no Custom Elements :(

### ///Custom Elements + Server Side Includes = ❤️

To make server rendering work, the previous example is refactored. Each team has their own express server and the `render()` method of the Custom Element is also accessible via url.

	$ curl http://127.0.0.1:3000/blue-buy?sku=t_porsche
	<button type="button">buy for 66,00 €</button>

The Custom Element tag name is used as the path name - attributes become query parameters. Now there is a way to server-render the content of every component. In combination with the `<blue-buy>`-Custom Elements something that is quite close to a **Universal Web Component** is achieved:

	<blue-buy sku="t_porsche">
	  <!--#include virtual="/blue-buy?sku=t_porsche" -->
	</blue-buy>

The `#include` comment is part of [Server Side Includes](https://en.wikipedia.org/wiki/Server_Side_Includes), which is a feature that is available in most web servers. Yes, it’s the same technique used back in the days to embed the current date on our web sites. There are also a few alternative techniques like [ESI](https://en.wikipedia.org/wiki/Edge_Side_Includes), [nodesi](https://github.com/Schibsted-Tech-Polska/nodesi), [compoxure](https://github.com/tes/compoxure) and [tailor](https://github.com/zalando/tailor), but for our projects SSI has proven itself as a simple and incredibly stable solution.

The `#include` comment is replaced with the response of `/blue-buy?sku=t_porsche` before the web server sends the complete page to the browser. The configuration in nginx looks like this:

	upstream team_blue {
	  server team_blue:3001;
	}
	upstream team_green {
	  server team_green:3002;
	}
	upstream team_red {
	  server team_red:3003;
	}

	server {
	  listen 3000;
	  ssi on;

	  location /blue {
	    proxy_pass  http://team_blue;
	  }
	  location /green {
	    proxy_pass  http://team_green;
	  }
	  location /red {
	    proxy_pass  http://team_red;
	  }
	  location / {
	    proxy_pass  http://team_red;
	  }
	}

The directive `ssi: on;` enables the SSI feature and an `upstream` and `location` block is added for every team to ensure that all urls which start with `/blue` will be routed to the correct application (`team_blue:3001`). In addition the `/` route is mapped to team red, which is controlling the homepage / product page.

This animation shows the tractor store in a browser which has **JavaScript disabled**.

[![server-render.gif](../_resources/78a0d87e0383b00b3b5bf49f404e5dc2.gif)](https://micro-frontends.org/ressources/video/server-render.mp4)

[inspect the code](https://github.com/neuland/micro-frontends/tree/master/2-composition-universal)

The variant selection buttons now are actual links and every click leads to a reload of the page. The terminal on the right illustrates the process of how a request for a page is routed to team red, which controls the product page and after that the markup is supplemented by the fragments from team blue and green.

When switching JavaScript back on, only the server log messages for the first request will be visible. All subsequent tractor changes are handled client side, just like in the first example. In a later example the product data will be extracted from the JavaScript and loaded via a REST api as needed.

You can play with this sample code on your local machine. Only [Docker Compose](https://docs.docker.com/compose/install/) needs to be installed.

	git clone https://github.com/neuland/micro-frontends.git
	cd micro-frontends/2-composition-universal
	docker-compose up --build

Docker then starts the nginx on port 3000 and builds the node.js image for each team. When you open http://127.0.0.1:3000/ in your browser you should see a red tractor. The combined log of `docker-compose` makes it easy to see what is going on in the network. Sadly there is no way to control the output color, so you have to endure the fact that team blue might be highlighted in green :)

The `src` files are mapped into the individual containers and the node application will restart when you make a code change. Changing the `nginx.conf` requires a restart of `docker-compose` in order to have an effect. So feel free to fiddle around and give feedback.

### ///Data Fetching & Loading States

A downside of the SSI/ESI approach is, that the **slowest fragment determines the response time** of the whole page. So it’s good when the response of a fragment can be cached. For fragments that are expensive to produce and hard to cache it’s often a good idea to exclude them from the initial render. They can be loaded asynchronously in the browser. In our example the `green-recos` fragment, that shows personalized recommendations is a candidate for this.

One possible solution would be that team red just skips the SSI Include.
**Before**

	<green-recos sku="t_porsche">
	  <!--#include virtual="/green-recos?sku=t_porsche" -->
	</green-recos>

**After**

	<green-recos sku="t_porsche"></green-recos>

*Important Side-note: Custom Elements [cannot be self-closing](https://developers.google.com/web/fundamentals/architecture/building-components/customelements#jsapi), so writing `<green-recos sku="t_porsche" />` would not work correctly.*

![data-fetching-reflow.gif](../_resources/7800bb5f7ab7de4addd334e4cd91198d.gif)

The rendering only takes place in the browser. But, as can be seen in the animation, this change has now introduced a **substantial reflow** of the page. The recommendation area is initially blank. Team greens JavaScript is loaded and executed. The API call for fetching the personalized recommendation is made. The recommendation markup is rendered and the associated images are requested. The fragment now needs more space and pushes the layout of the page.

There are different options to avoid an annoying reflow like this. Team red, which controls the page, could **fixate the recommendation containers height**. On a responsive website its often tricky to determine the height, because it could differ for different screen sizes. But the more important issue is, that **this kind of inter-team agreement creates a tight coupling** between team red and green. If team green wants to introduce an additional sub-headline in the reco element, it would have to coordinate with team red on the new height. Both teams would have to rollout their changes simultaneously to avoid a broken layout.

A better way is to use a technique called [Skeleton Screens](https://blog.prototypr.io/luke-wroblewski-introduced-skeleton-screens-in-2013-through-his-work-on-the-polar-app-later-fd1d32a6a8e7). Team red leaves the `green-recos` SSI Include in the markup. In addition team green changes the **server-side render method** of its fragment so that it produces a **schematic version of the content**. The **skeleton markup** can reuse parts of the real content’s layout styles. This way it **reserves the needed space** and the fill-in of the actual content does not lead to a jump.

![data-fetching-skeleton.gif](../_resources/8c57df2888cf14670f04fdc165ce4d23.gif)

Skeleton screens are also **very useful for client rendering**. When your custom element is inserted into the DOM due to a user action it could **instantly render the skeleton** until the data it needs from the server has arrived.

Even on an **attribute change** like for the *variant select* you can decide to switch to skeleton view until the new data arrives. This ways the user gets an indication that something is going on in the fragment. But when your endpoint responds quickly a short **skeleton flicker** between the old and new data could also be annoying. Preserving the old data or using intelligent timeouts can help. So use this technique wisely and try to get user feedback.

## //Navigating Between Pages

**to be continued soon…**

watch the [Github Repo](https://github.com/neuland/micro-frontends) to get notified

## //Additional Resources

- [Talk: Micro Frontends - YGLF, Tel Aviv 2017](https://www.youtube.com/watch?v=o1Sr39DVdOQ) ([Slides](https://speakerdeck.com/naltatis/micro-frontends-yglf-tel-aviv))

- [Slides: Micro Frontends - JSUnconf.eu 2017](https://speakerdeck.com/naltatis/micro-frontends-building-a-modern-webapp-with-multiple-teams)

- [Talk: Break Up With Your Frontend Monolith - JS Kongress 2017](https://www.youtube.com/watch?v=W3_8sxUurzA) Elisabeth Engel talks about implementing Micro Frontends at gutefrage.net

- [Post: Micro frontends - a microservice approach to front-end web development](https://medium.com/@tomsoderlund/micro-frontends-a-microservice-approach-to-front-end-web-development-f325ebdadc16) Tom Söderlund explains the core concept and provides links on this topic

- [Post: Microservices to Micro-Frontends](http://www.agilechamps.com/microservices-to-micro-frontends/) Sandeep Jain summarizes the key principals behind microservices and micro frontends

- [Link Collection: Micro Frontends by Elisabeth Engel](https://micro-frontends.zeef.com/elisabeth.engel?ref=elisabeth.engel&share=ee53d51a914b4951ae5c94ece97642fc) extensive list of posts, talks, tools and other resources on this topic

- [Custom Elements Everywhere](https://custom-elements-everywhere.com/) Making sure frameworks and custom elements can be BFFs

- Tractors are purchasable at [manufactum.com](https://www.manufactum.com/) :)

*This store is developed by two teams using the here described techniques.*

* * *

## //Things to come …

- Use Cases

    - Navigating between pages

        - soft vs. hard navigation

        - universal router

    - …

- Side Topics

    - Isolated CSS / Coherent User Interface / Style Guides & Pattern Libraries

    - Performance on initial load

    - Performance while using the site

    - Loading CSS

    - Loading JS

    - Integration Testing

    - …

## //Author

**Michael Geers** ([@naltatis](https://twitter.com/naltatis)) is a software engineer at [neuland Büro für Informatik](https://www.neuland-bfi.de/) and works on building nice frontends for e-commerce.

This site is generated by Github Pages. Its source can be found at [neuland/micro-frontends](https://github.com/neuland/micro-frontends/).

 .