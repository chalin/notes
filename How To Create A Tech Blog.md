How To Create A Tech Blog

## How To Create A Tech Blog

[Torgeir "Tor" Helgevold](http://www.syntaxsuccess.com/bio)

- JavaScript Developer and Blogger

##### Published: Mon Feb 20 2017

Blogging is one of my favorite hobbies. In this post I want to share some info about how I built up my blog. Hopefully this will be helpful to other bloggers, but I am also looking for feedback and suggestions for improvement.

#### How I got started

When I started my blog I was mainly viewing it as a way to document my JavaScript experiences. My hope was that it would make it easier to look up technical reference material if I ever needed it again in the future.

Initially, sharing my material with the world was secondary at best. However, as traffic to my site started to increase, I became convinced that my material was useful to others as well. That's when I decided to increase the online visibility of my articles.

#### SEO

About 70% of my traffic is from organic search via Google, or other search engines. Since I don't have a big social media following, good SEO is key to getting the word out about my content.

Much of my content describes challenges I've faced and solved in my day job. This means it by default answers real world problems that other developers face on a daily basis.

The secrets behind the google indexing algorithm is not public knowledge. I do think the document title plays a role though. Before picking an article title I generally ask myself: “What would I type into google if I am searching for this type of content?”

Right after publishing a new article I log in to Google web master's tools and request indexing of the new article.

I also add a few meta tags like url and description. Not sure if this matters for SEO, but at least it is used by social media sites like Facebook and Twitter.

So far this has worked out well since I often see my articles on the first or second search result page in Google.

#### Crawlers

It's important to serve your pages using crawler friendly technology. While modern crawlers are able to execute some JavaScript, your site is at a huge disadvantage if use a JavaScript framework to render articles. You are much better off switching to server side rendering of your articles.

If you are using an existing blogging platform like Medium or Wordpress you don't have to worry about these things. I have built my blog from scratch since it gives me a better opportunity to play with different technologies. It also gives me full control end-to-end. I depend on this whenever I deploy live demos to support my articles.

The downside is that you are on your own when it comes to graphic design and styling of the blog. I am not known for css skills, so my site suffers a bit from less than optimal styling.

#### Social Media

Social media platforms like Twitter, Facebook and LinkedIn are great avenues for sharing your articles. I recommend sharing your own content directly. Over time you should share the same articles multiple times.

Facebook and LinkedIn let you join developer groups for specific topics (Angular, JavaScript, etc.). These groups are great for sharing articles since the members are by default interested in content that matches the group.

Twitter is another great way to share articles. You may have a modest following, but adding good hashtags expands your audience beyond your own following.

It takes time to build up a following on Twitter, but give it time. I am still working on mine.

In addition to self sharing, you should also add social media sharing widgets next to articles. This will enable your readers to share your content with their potentially big social media following.

#### Engage Other Experts

If you have written an insightful article about a particular topic, you may want to reach out to other “experts” and point them to your article.

There are a few different ways you can do that. Twitter or Blog comments are a few examples. You have to walk a fine line here though. I don't think it's a good idea to comment on someone else's blog with something that is nothing more than a shameless plug.

It's best to only link to your content if it actually adds value.

#### Don't wait for perfection

A lot of people are perfectionists. As a result they avoid publishing their blog post until they think it's “perfect”. I think this is a mistake. I find that its better to publish your article even though it's not perfect yet. Remember, you can always go back and revise your content later.

If it takes you several days to publish an article, you may be overthinking things.

I also try to limit the length of my articles. Short and to the point content is usually best in my opinion.

#### Analytics

Google Analytics is free to add to your blog. It gives you great insight into how users are interacting with your content. It can be fun to see which articles get the most love from your readers.

Personally I am not optimizing my content based on past articles. To me the most important part is writing about things that interest me.

A perfect example of this is all my recent content about bundling and the Closure Compiler. These are not topics that interest the masses, but it's is something I really enjoy personally. Enjoying what you write about is much more important than number of views in my opinion.

#### Hosting

There are many alternatives for hosting. I have deployed my blog as a nodeJS/Express site in Azure. I am able to get by with a modest server in their shared hosting tier. The total cost is somewhere around $11/month, which I think is acceptable.

Initially I was worried the server would buckle under load, but I have seen loads as high as 182 concurrent users as seen in the screenshot below:

![](../_resources/525a83cb794e81debed509842123ede4.png)

There are however memory and cpu constraints in Azure's shared tier that will cause your site to be suspended if you go above the quota.

#### Traffic

It takes time to get to a point where your blog gets a substantial amount of traffic. I have been blogging for about two years now, and during a good month I've seen my traffic get up to 95k page views. This is of course nothing compared to some of the more prominent tech blogs out there.

That said, I am still proud of my progress so far. After all I am just a hobby blogger who does this for fun on the side :-).

If you liked this article,

[**Tweet](https://twitter.com/intent/tweet?original_referer=http%3A%2F%2Fwww.syntaxsuccess.com%2Fviewarticle%2Fhow-to-create-a-tech-blog&ref_src=twsrc%5Etfw&text=How%20To%20Create%20A%20Tech%20Blog&tw_p=tweetbutton&url=http%3A%2F%2Fwww.syntaxsuccess.com%2Fviewarticle%2Fhow-to-create-a-tech-blog)

it to your friends.

I also invite you to follow me on twitter

[**Follow **@helgevold**](https://twitter.com/intent/follow?original_referer=http%3A%2F%2Fwww.syntaxsuccess.com%2Fviewarticle%2Fhow-to-create-a-tech-blog&ref_src=twsrc%5Etfw&region=follow_link&screen_name=helgevold&tw_p=followbutton)