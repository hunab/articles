#Tuenti UI Coding Style Guide
*Written on July 2012.* 

##General guides

At Tuenti we love HTML and CSS, but we also know it can cause lots of problems. That's why we use some of the following practices to keep our "love affair" going strong. This article is not a list of recommendations, but rather our way of sharing some guidelines used at Tuenti. I was inspired to share our methodology after reading [Harry Robert's article](http://csswizardry.com/2012/04/my-html-css-coding-style/) (kudos Harry).

Tuenti is a big project with lots of code, so you can imagine how hard is to work in a collaborative environment where different people can create or modify HTML and CSS, and where, above all, you need to provide the most efficient solutions possible. To accomplish that, we follow a [modular oriented approach](https://github.com/stubbornella/oocss/wiki) to generate reusable components and to mantain a consistent UI components framework.

Using this modular approach has some key benefits:

* Write less CSS, and more predictable (because we maximize 1:n relation between CSS vs HTML)
* Less browser rendering problems (because we use more previously tested code)
* Easier to mantain
* More flexible

##HTML

You need to be flexible with your markup to provide efficient components, I remember in the past trying to avoid "divitis" and "classitis" all the time, and mantaining the markup as clean as possible. However, [these practices are bad](http://www.slideshare.net/stubbornella/our-best-practices-are-killing-us) when associated with modularity; the markup is too closely coupled with the CSS, resulting in less flexibility and more troublesome maintenance.

We try to create modules that can adapt to different HTML structures, achieved by applying different classes to our HTML, so our HTML is happily affected by "classitis", "divitis"; and others not-so-effective called "good practices".

Additionally, we write our markup always in lowercase, to mantain coherency throughout our platform, which improves the readability and consistency of our code. We also focus on readability by adding blank lines between some structures, and comments in some closing tags.

###Tags

We always close our tags. Although we know there are some situations where you can avoid closing tags, we've decided to explicitly close all tags to avoid confusing other developers. If I were coding for a personal project, I would probably ommit some closing tags (specially li's or td's). But efficient teamwork requires predictable code.

We also use certain tags for some specific things. For example we use `<i class="i-photo"></i>` for icons and and `<b></b>` for text in buttons or to accompany icons.

###Attributes

We always use quotes for attributes, again for the sake of consistency. This means if you're using an OOCSS approach, you'll often use multiple classes, so it's better to always to quote attributes.

Recently, we've started using dashes to separate attribute words. We previously used a camel case approach, but have learned that dashes improve readability. Here at Tuenti, the markup you write will always be read by your colleagues, so we make an effort to follow conventions.

##CSS

Following the OOCSS approach, we use classes in order to abstract modules. This is a great way to avoid [code repetition](http://en.wikipedia.org/wiki/Don't_repeat_yourself) as the project grows. Be careful with the abstractions. Although coupling HTML and CSS is a big problem, abstracting too much can also generate issues (like having to apply too many classes to an element to style it). Try to separate concepts like structure, appearance, and function.

We have some basics CSS files (reset, layout and structures) and lots of small CSS files for every module, and then we concatenate and minify them in the server side. In fact this separation is great for loading only the modules you need… but that's another story. Working in a CSS file with thousands of lines can be a real mess, so don't be afraid of having to deal with lots of small files.

Here’s a summary of the practices used in our modular approach:

* Avoid id's for styling hooks (mainly because of the specificity)
* Avoid over-qualified classes (p.whatever, because of flexibility and specificity)
* Short selectors (maximum 3 or 4 classes, because of the specifity, readability and performance)
* Separation of container and content (`.bulleted-list` instead of `#sidebar ul` for exameple)
* Try always to use our grid framework to build module structures
* Avoid !important and inline styles

This approach tries to minimize the "C" in CSS (cascading) by only using cascading on small components that do not affect other structures. Again, this means that lots of good practices we used in the past are not advisable to create modular CSS.

###Naming classes

We use different naming conventions to easily predict which styles to apply, for example we use `.h-???` to create general helpers:

	.h-right { float: right; display: inline; }
    .h-left { float: left; display: inline; }
    .h-pr { position: relative; }
    .h-ir { font: 0/0 a; text-shadow: none; color: transparent; }
    ...
    
We create components (like `.item`) and then we namespace the elements inside the component, usually with the first three consonants (`.itm-media`, `.itm-body`). We even leave the component root empty to increase the readability:

    .item {}
        .itm-media { float: left; display: inline; margin-right: 16px; }
            .itm-media img { display: block; }
        .itm-body { display: table; zoom: 1; }
        .itm-body-exp { display: block; overflow: hidden; }

###Selectors

As I mentioned above, we try to keep our selectors as short as possible to maximize portability and minimize dependency and specificity – this avoids the specificity wars of the past. The way you write your selectors is crucial to avoid out-of-control style sheets, so you must write them carefully.

The most important part of a selector is the key selector due to performance reasons ([key selector is the last part of a selector](https://developer.mozilla.org/en-US/docs/CSS/Writing_Efficient_CSS?redirectlocale=en-US&redirectslug=Writing_Efficient_CSS)), because browsers read selectors from right to left. So we try to write key selectors that are class selectors instead of type selectors (an element like p or li).
        
To increase readability, we also indent related selectors to see the hierarchy easily, and we add some white space between blocks.

    .header { ... }
        .hdr-form { ... }
            .hdr-search { ... }
        .hdr-actions { ... }
        
    .footer { ... }
        .ftr-lang { ... }

###Properties

We order CSS properties by relevance (position and size first, then margin, padding, fonts, colors, and finally everything else). We don't have an explicit rule for this, we prefer common sense. To increase readability, we like to add whitespace after ":" and align vendor prefixes around ":". We also use one line per property if the block of properties has more than 4 properties. This creates a nice balance between increased readability and proper function with version control tools. Here's an example:

	.itm-body { display: table; zoom: 1; }
    .itm-actions { 
        position: absolute; 
        top: -6px;
        right: 16px;
        opacity: 0; 
        filter: alpha(opacity=0);
        -webkit-transition: .3s ease-out;
           -moz-transition: .3s ease-out;
             -o-transition: .3s ease-out;     
    }

###Comments

When writing CSS, is important to use comments appropiately. They aren't just to explain some selector or some property, but also help to:

* Create a nice index for your content
* Add titles to easily differentiate your modules
* Show HTML for a given module to help other developers
* Even to [generate automatic documentation with your css coments](http://jacobrask.github.com/styledocco/) with your CSS comments

##Wrapping up

I hope you've enjoyed these "Tuenti practices". Take what makes sense for you and adapt it to your needs. But remember, the "best practices" we used in the past are now obsolete. So never stop innovating and evolving!