---
title: "A pure CSS burger menu"
description: "No JS, works on mobile!"
date: "2022-03-01"
timezone: "Europe/London"
utterances_term: "pure-css-burger"
categories: [HTML, CSS, responsive]
---
# A pure CSS burger menu

> No JS, works on mobile!

Just because it's interesting, let's make a burger menu, without using JS to handle hiding/showing the menu content.

What're we going to do?

1. Set up a nav menu.
2. Move that menu off the side of the viewport, so it can't be seen.
3. Add a hamburger icon to that nav menu that will remain on screen. When we hover (or tap on mobile), we'll bring the menu back onto the screen, and hide the hamburger.

First, let's lay out our menu content:
```html
  <nav>
    <div class="menu-content">
      <img 
        src="http://tiny.cc/em2puz"
        alt="Unsplash is fantastic.  Check them out at the img src." 
        class="nav-logo"
      />
    <ul>
      <li><a href="#">Link 1</a></li>
      <li><a href="#">Link 2</a></li>
      <li><a href="#">Link 3</a></li>
    </ul>
    </div>
    <h2 class="burger-button">❰❰</h2>
  </nav>
```

It's nice and simple to use a couple of unicode chevrons for our burger menu, but we could swap anything we like into the "burger-button" element's position.  As long as we can control its width.
> We don't actually need to control its width, but doing so reduces the trial-and-error necessary for getting the hamburger icon positioned neatly.

We can put anything we like in the menu, but it's useful to have an element with a known width (and margin and padding) defining the overall width of the menu, so we can move it around accurately.

Speaking of styling, let's set some basic dimensions.  Using rem as a unit is helpful for resizing things basic on a single code unit (the base text size).  We're using two text characters for our logo, so we can set its font size, and know that the "logo" is `font size * 2` wide.

```css
:root{
  --nav-logo-width: 5rem;
  --nav-logo-margin: 1rem;

  --burger-icon-width: 2rem;
}
.nav-logo{
  width: var(--nav-logo-width);
  margin: var(--nav-logo-margin);
}
.burger-button{
  font-size: var(--burger-icon-width);
}
```

Next up, let's get our nav bar stacked up neatly, and over to the beginning of our screen:
Rtl rendering breaks this thoroughly at the moment, so I'll be sticking with left and right from here on.
```css
nav{
  display: flex;
  position:fixed;
}
```

#### Great!  We've got a menu laid out, with an icon next to it.

![basic nav layout](https://user-images.githubusercontent.com/72463218/156226555-95502eb1-9f4b-4e3f-8e05-ed36e53e78e5.png)

Now, let's shove our menu off the screen:
```css
nav{
  right: calc(100vw - var(--burger-icon-width));
}
```
Nice and easy:
1. Push the whole nav element over until its right hand side (`right:`) is the entire view port away from the right hand side of the screen (`100vw`)
1. Bring it back by the width of our burger icon (`var(--burger-icon-width)`)

Finally, when we hover our logo (or tap it on mobile, which engages the `:hover` pseudo selector), let's bring the nav menu back (and hide the burger button).  
> We need to make sure that we don't move the nav element too far from the side of the screen, or the `:hover` state will be disengaged as soon as it engages, and we'll be stuck with a nav bar flashing between the two states.  Not pretty!

> Handling of :hover, :focus, and :active states has changed a lot over the last few years.  Check out https://bitsofco.de/when-do-the-hover-focus-and-active-pseudo-classes-apply/ for an interesting discussion.

```css
nav:hover{
  right: unset;
}
nav:hover .burger-button{
  display: none;
}
```
Clean again!

1. Unset the `right` property on the nav menu, to bring it back to its default `fixed` position.
1. With the hover state active on the nav menu, hide the burger button.

#### All done!




Now, let's get it looking tolerable.

Let's give our menu a background colour so we can see it over our content, and z-index it to make sure of that.
```css
:root{
  --color-bg: lightgray;
}
nav{
  z-index: 10;
}
nav .menu-content{
  background-color: var(--color-bg);
}
```

Reduce opacity of our hamburger, so we can see our content behind it whilst keeping all of our screen real estate:

```css
.burger-button{
  opacity: 0.6;
}
```


### And bring it all together:
https://codepen.io/dbbrowne/pen/NWwEXGo


Things work for now, but some some easing of the movements would be nice.  And a slightly prettier menu.

What we'd like is to just put an animation on the movement as the navbar slides in.  Something like:
```css
:root{
  --nav-transition-duration: 0.5s ease;
}
nav{
  transition: right var(--nav-transition-duration);
}
```
But unfortunately, it's not that simple.  Transitions only seem to work if the CSS has a defined position to work to/from.  Restoring our nav bar with `right: unset;` breaks this.  
We need to replace our `nav:hover` styling:
```css
nav:hover{
  right: calc(100vw - var(--nav-logo-width) - var(--nav-logo-margin)*2);
}
```

We need to keep the property we use for positioning consistent to use `transition`, so we need to define the "on-screen" position relative to `right`.  So, assuming that the logo is the width-defining feature of the nav bar..
1. `100vw` - 100% of the viewport away from the right hand side...
1. `- var(--nav-logo-width)` - less one logo width...
1. `- var(--nav-logo-margin)*2` - and less both margin widths...
1. any other adjustments for padding/margins given our specific situation.

We might make life easier here by fixing the width of the nav bar directly, and then sizing its contents to suit.  Up to you!


There are still a few things to think about - accessibility and screen readers may not be well served by this approach.  Please give me some advice in the comments about how to handle this.


We can also make our nav bar a little slicker.  If we're using something like *React Router*'s [NavLink](https://v5.reactrouter.com/web/api/NavLink), then the currently active link will have an `active` class.  If we then move our burger icon into the link, we can give our user an indication of where they are:
``jsx
<NavLink to="url">
  <span>link title</span>
  <LeftArrow />
</NavLink>
```
```css
a.active{
  background-color: var(--nav-color-active);
}
```

<img src="https://user-images.githubusercontent.com/72463218/156239739-ba76eefc-d217-49ba-8556-0d7688e9f92e.gif" alt="navbar demo" width="200"/>


Check out the latest version at dbb.tools, or as it is in the animation above [on github](https://github.com/DBBrowne/portfolio/blob/66938b3402a0e4dd4e504aa271b84baa56d1cd87/src/containers/common/Nav.js#L37-L38).

Thanks for your time, please do offer advice in the comments, or over LinkedIn!