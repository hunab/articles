Title:  Our love/hate story with modern CSS
Author: Marcio Barrios
Email:  marciobarrios@gmail.com
Date:   July 2013

# Our love/hate story with modern CSS
*Written in July 2013 for Tuenti dev blog*

Last year in the Tuenti design team we had the opportunity to rethink and then remake all the UI for a new Tuenti. After lot of research, prototypes and designs we coded our UI framework to generate reusable components and to be more flexible and mantainable than we were in the past.

We decided to drop the support for IE7, and that was a great decision, that aimed us to use less hacky and more standard CSS, with all the benefits of using newer selectors, and better CSS3 features like transitions or animations. And let's be honest, CSS3 is funny to use and makes our work more enjoyable.

Of course not every site has the same performance issues and some of the things I'm going to expose won't be useful to you, but it's always good to know the potential issues and how to avoid them.

## Something was wrong when scrolling...

The main thing that stands out a problem with fluency in our interface is scrolling, so it was quite easy to test if our interface was fast enough scrolling like crazy across the site, specially in lists where we gathered large amounts of items, in our case the main hot spot is the chat buddy list, when you can see all your friends and start a conversation with them. The chat is the most important feature for our users, we serve millions of messages a day, so the experience using the chat is vital for our success.

We spotted a problem when scrolling in modern browsers for users with many friends, it was sluggish, so you can imagine the experience using IE8 or even IE9, it was a very bad experience.

After that, we started to figure out what the hell was happening if we were using clean and fancy CSS. Oh! probably that was the problem...

## Discovering the potential issues

The responsiveness and smoothness of the UI affects user experience, and that means we need to mantain our UI as close as possible to [60fps (frames per second)](http://www.smashingmagazine.com/2013/06/10/pinterest-paint-performance-case-study/) when scrolling, or at least above 30fps.

As a summary, take into account these 4 concerns about CSS performance to achieve the best frame rate possible, in order of more to less expensive:

- **Reflow** (or relayout), happens when manipulating the DOM, changing a style that affects the layout, resizing the window, changing the font, activating the :hover... summarizing, everything that cause layout changes (padding, border, margin...).
- **Repaint** (or redraw), happens when there are changes that are made to an elements skin, but do not affect its layout (background-color, color, visibility, outline...)
- **Style recalculation** (chrome timeline)
- **Selector matching** (chrome profiler)

The theory is simple, try to minimize the number of reflows/repaints to improve the performance. All these concepts deserve an entire article to explain it carefully, I recommend you to read this [article from Addy  Osmani](http://addyosmani.com/blog/devtools-visually-re-engineering-css-for-faster-paint-times/), a detailed explanation of the concepts and its implications.

Here a list of potential issues that could cause expensive reflows/repaints:

- [border-radius and box-shadow][borderboxshadow]
- [Slower selectors][slowerselectors]
- [Fixed position][fixedposition]
- [Moving elements][movingelements]
- [Lots of DOM nodes on scrolling areas][toomuchnodes]
- [Hidden elements][hiddenlnodes]

### border-radius and box-shadow properties [borderboxshadow]

It's very common the use of these properties, but are not easy to render for the browser, and if you abuse using it your scroll won't be smooth, try to use it with caution. In the case of box-shadow don't use it on big elements or fixed ones, and try to minimize as much as you can the spread value of the box-shadow (Airbnb had a problem with box-shadows, you can [read their experience](http://nerds.airbnb.com/box-shadows-are-expensive-to-paint/)).

In fact, the combination of both have a greater paint time than the sum of their parts, so it's a perfomance killer. You can read here more info about [CSS paint times](http://www.html5rocks.com/en/tutorials/speed/css-paint-times/).

###  Slower selectors [slowerselectors]

Selectors are not slow by default, but some of them could be very slow when matching large amount of elements, specially large lists with scroll (this is the case of our chat buddy list). So if you use [general sibling selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/General_sibling_selectors) (~) or [adjacent sibling selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Adjacent_sibling_selectors) (+) in one of these extreme cases it's a bad choice, it's much better in terms of performance to use a simple class in every item because the selector matching it's much faster for the browser.

Another case of potential slow selectors are [attribute](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) and [pseudo-elements](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-elements) selectors, of course you can use them and in fact are very useful in some situations, but don't abuse, according to the book [Even Faster Websites](http://www.amazon.com/Even-Faster-Web-Sites-Performance/dp/0596522304) of Steve Souders (a guru of the topic), pseudo-elements are the worst selectors in terms of performance, here the complete list in order of more to less efficient selectors:

1. ID, e.g. *#header*
2. Class, e.g. *.promo*
3. Type, e.g. *div*
4. Adjacent sibling, e.g. *h2 + p*
5. Child, e.g. *li > ul*
6. Descendant, e.g. ul a
7. Universal, *
8. Attribute, e.g. *[type="text"]*
9. Pseudo-classes/-elements, e.g. *a:hover*

And remember, the most important part of a selector is the key selector ([key selector is the last part of a selector](http://csswizardry.com/2011/09/writing-efficient-css-selectors/)) because browsers read selectors right-to-left.

### Fixed position [fixedposition]

Fixed content can be problematic because of repaints happening in the page when you scroll, later I'll show you how to detect long paint times.

So it's good to know that `position: fixed` and `background-attachment: fixed` can cause an extra painting cost.

### Moving elements [movingelements]

Transitions or animations cause reflows and repaints in the page, so it's expensive in order to achieve the 60fps, try to not add box-shadows to animated elements to not increase exponentially the cost. And always use `translate` instead of `top/right/bottom/left` properties to [take advantage of GPU acceleration](http://www.paulirish.com/2012/why-moving-elements-with-translate-is-better-than-posabs-topleft/), the animation will be smoother.

### Lots of DOM nodes on scrolling areas [toomuchnodes]

It's important to keep the number of nodes to a minimum specially in scrolling areas, because the selector matching and the styles could affect the perfomance, specially if we use a combination of lots of nodes and slow selectors like we mentioned before.

### Hidden elements [hiddenlnodes]

We used to hide lots of elements in the chat buddy list in order to show them when the user hovered or clicked in a specific element. That's definetely a bad practice, because these elements are being matched with the corresponding styles, and that's an unnecessary penalty for the perfomance.

## Useful tools and resources to detect and resolve performance issues

The main and the most important tool is inside Chrome browser, [Chrome Dev Tools](https://developers.google.com/chrome-developer-tools/) is a powerful help to detect performance issues, let's see how.

### Timeline panel (Frame Mode)

In the Timeline tab of Chrome Dev Tools you have a valuable tool to check the performance of your site, you can view the frame rate when you scroll, so you would see some peaks in this panel if your site has any performance issue. Steps to use it:

1. Record some activity (with the circle button in the bottom bar).
2. Analyze the bars at the top, looking for peaks (the smaller bars the better).
	- There are 2 horizontal lines, one at 30fps and other at 60fps.
	- Every bar is composed from different colors, the most important one for us is the green one, that is the paint time.

![Timeline screenshot](timeline-frames.png?raw=true)

For a deeply article about Timeline [read the documentation page](https://developers.google.com/chrome-developer-tools/docs/timeline).

### Profiles panel

The profile panel will show you how many DOM elements are matching with your CSS selectors and how long this matching has taken, this is quite useful because you can find some peaks and try to reduce it. 

Mantaining your selector matching low will increase your site performance.

Full documentation for [Profiles panel](https://developers.google.com/chrome-developer-tools/docs/profiles).

### Other Chrome dev tools utils

#### Audits panel

With the Audit panel you can analyze how a web page loads and receive suggestions and optimizations to improve page load and responsiveness.

Related with this topic, Google has a Chrome extension called [PageSpeed](https://developers.google.com/speed/pagespeed/) to help you with more detailed information.

#### Settings

There are several options in settings panel to help you detect some performance issues, here I show you some useful ones but check the [full list of settings](https://developers.google.com/chrome-developer-tools/docs/settings):

* **Continuous painting mode:** you will see a small graph similar to the one in Timeline tab, so it will help you to identify bottleneck on your page. You can change styles on the fly to figure out what is causing the slowness.
* **Show paint rectangles:** allows you to see the regions where Chrome paints, this feature helps you to [avoid unnecessary paints](http://www.html5rocks.com/en/tutorials/speed/unnecessary-paints/) and [study painting behaviors](http://www.paulirish.com/2011/viewing-chromes-paint-cycle/) on the page.
* **FPS meter:** shows a layer at the top right border with information about the frame rate, very useful to check if some animation is affecting the FPS.

### Other useful tools

* [Web Tracing Framework from Google](http://google.github.io/tracing-framework/), a recent released collection of tools for instrumenting, analyzing, and visualizing web apps.
* [Virtual machines](http://www.modern.ie/en-us/virtualization-tools#downloads) to test different versions of IE.
* [BrowserStack](http://www.browserstack.com/), an online service that let's you test lots of versions of different browsers in multiple OS.

## How we fixed the smoothness in the chat list

Based on the information in the article, we did this in order to improve the perfomance:

1. Simplify style: we got rid of box-shadow and gradients in list items, it was a problem when scrolling.
2. Rewrite slow selectors (~, + and pseudo-elements), because in combination with big amount of nodes made the UI sluggish.
3. Remove almost all hidden elements from every item, that increased a lot the selector matching.
4. Refactor the HTML/CSS to use less nodes, for example the way we managed the icon states:

Before:

	<li>
		<span title="Options" class="icon i-states">
			<i class="icon i-options i-default i-op-hard">Options</i>
			<i class="icon i-options i-hover i-op-dark"></i>
			<i class="icon i-options i-active i-op-dark"></i>
		</span>
	</li>

After:

	<li title="Options" class="icon i-options i-default-hard i-hover-dark i-active-dark">Options</li>

Think about the situation of 200 or 300 users in the chat list, if you multiply the nodes we removed (around 10 nodes per item), we saved 2000 nodes! and that plus the better selector matching and a better framerate without box-shadows and gradients made a big difference, finally the chat list was smooth when scrolling.

## Reflexion about CSS performance

Like I said in the beginning, not every site is affected by these performance problems even if you don't use good practices or don't take perfomance into account, because performance problems are mainly related with the combination of scroll + high number of nodes + slow properties.

One important challenge is to increase perfomance in mobile devices, we already have good tools to remote debugging these devices, but this topic is quite difficult, there are lots of different devices with different capabilities.

As a summary, try to be aware of what things make your site slower, and measure performance bottlenecks when you're building it to try to avoid future performance issues.

I hope you help it!