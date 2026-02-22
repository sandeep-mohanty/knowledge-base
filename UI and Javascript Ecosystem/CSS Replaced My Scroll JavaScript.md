# CSS Replaced My Scroll JavaScript (Real UI)
**Modern scroll UI without JavaScript (Real-World Examples)**

For years, scroll-based UI meant one thing: **JavaScript everywhere**. Scroll listeners. Throttling functions. Position calculations. Class toggles. Hoping it doesn’t lag on mobile.

Then I discovered something that genuinely surprised me: **Modern CSS can now react to scroll state natively.** Not scroll position hacks. Not magic libraries. Actual built-in browser logic.

I built a full demo page using **zero scroll JavaScript** and everything still works. Let me show you what changed.

---

## The Big Change: CSS Can Now Watch Scroll State
Here’s what we used to have: `window.scrollY`. A number. We’d read it, do math, and make decisions.

Here’s what we have now: **CSS container queries for scroll state.**



Modern CSS just unlocked the ability to know:
* Can this container scroll up or down?
* Is this sticky element currently stuck?
* Is this slide the active snapped item?

Instead of writing JS to detect that, we just write CSS. The browser handles the rest.

---

## Example 1: A “Back to Top” Button With No JavaScript
Normally, you'd add a scroll listener, check if the page scrolled past 500 pixels, and toggle a class.

**Here’s the CSS-only way:**
```css
html {
  container-type: scroll-state;
  container-name: page;
}

@container page scroll-state(scrollable: top) {
  .back-to-top {
    translate: 0 0;
    opacity: 1;
  }
}
```
When the page can scroll upward (meaning we’re not at the top), the button appears. No event listeners. No throttling. No math.

---

## Example 2: Table of Contents That Highlights as You Scroll
Normally, I’d use `IntersectionObserver` to detect which section is visible, then manually update which link is active.

**Not this time:**
```css
.toc-list {
  scroll-target-group: auto;
}

.toc-list a:target-current {
  color: #3b82f6;
  font-weight: 600;
}
```
As I scroll, the link for the visible section automatically highlights. The browser already tracks this for anchor navigation; `:target-current` just lets us style it.



---

## Example 3: Sticky Header That Knows When It’s Stuck
We've all wanted to add a shadow or background to a sticky header *only* once it actually sticks to the top.

**Now it's easy:**
```css
.site-header {
  position: sticky;
  top: 0;
  container-type: scroll-state;
  container-name: header;
}

@container header scroll-state(stuck: top) {
  .header-content {
    background: white;
    box-shadow: 0 4px 20px rgba(0,0,0,.1);
  }
}
```
The browser already knows when the header is stuck. Now we can style it based on that state without measuring anything.

---

## Example 4: A Pure CSS Carousel
I built a working carousel with smooth scrolling, navigation dots, and arrow buttons—all with **zero JavaScript**.

**Step 1: The scroll container**
```css
.carousel {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  scroll-behavior: smooth;
}
```

**Step 2: Let the browser create the buttons**
```css
.carousel {
  scroll-marker-group: after;
}

.carousel-slide::scroll-marker {
  content: "";
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: #ccc;
}

.carousel::scroll-button(left)  { content: "←"; }
.carousel::scroll-button(right) { content: "→"; }
```

**Step 3: Style the active slide**
```css
.carousel-slide {
  container-type: scroll-state;
  container-name: slide;
}

@container slide scroll-state(snapped: x) {
  .carousel-slide img {
    opacity: 1;
    scale: 1;
  }
}
```



---

## Why This Actually Matters
It’s not just a trick; it’s a performance upgrade.
* **It’s Faster**: JS scroll listeners run on the main thread. CSS queries run in the browser’s layout engine. You’re using work the browser is already doing.
* **Less Main Thread Work**: Smoother scrolling and better responsiveness.

### When You Still Need JavaScript
CSS doesn't replace everything. You still need JS for:
* Infinite scroll with API calls.
* Complex animations with precise timing.
* Business logic (analytics, lazy loading).

---

## Conclusion
The future of scroll-based interfaces looks a lot more stylesheet-driven. Once you see these native features working, it’s hard to go back to heavy scroll listeners for simple UI effects.
