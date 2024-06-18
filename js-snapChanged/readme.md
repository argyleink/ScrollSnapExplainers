# JS Snap Events

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

CSS scroll snap points are often used as a mechanism to create scroll interactive "selection" components, where selection is determined with JavaScript intersection observers and a scroll end guestimate. By creating a built-in event, the invisible state will become actionable, at the right time, and always correct.

## Goals

Help developers sychronize a snapped scroll item with the rest of their interface elements efficiently and effectively.

<br>

## Use Cases

Scroll centric selection:
- Date / Time picker
- Carousel
- Tabs
- State / Country picker

<br>

## Example

In a scroll-snapping carousel of images, a developer might want to style or animate the
image that the carousel is snapped to differently from other images in the
carousel or perform some other action on the page in sync with what is snapped to.

For example, given the following HTML:

```
<style>
  #snapcarousel {
    ... /* Style creating horizontal scrolling carousel. */
    scroll-snap-type: x mandatory;
  }
  .snaparea {
    ... /* Style creating horizontal scrolling carousel. */
    scroll-snap-align: center;
  }
</style>

<div id="snapcarousel">
  <div id="image1" class="snaparea"><img src="..."></img></div>
  <div id="image2" class="snaparea"><img src="..."></img></div>
  <div id="image3" class="snaparea"><img src="..."></img></div>
  <div id="image4" class="snaparea"><img src="..."></img></div>
  <div id="image5" class="snaparea"><img src="..."></img></div>
</div>

<div id="snapcarouselctrl">
  <div id="button1" class="imgbutton"></div>
  <div id="button2" class="imgbutton"></div>
  <div id="button3" class="imgbutton"></div>
  <div id="button4" class="imgbutton"></div>
  <div id="button5" class="imgbutton"></div>
</div>
```
where `snapcarousel` contains a set of images which can be viewed by
scrolling horizontally and `snapcarouselctrl` gives an alternative way to scroll
`snapcarousel` via buttons which, when clicked, scrolls their associated image
to `snapcarousel`'s center.

A developer might want to synchronize the styles of both the
image selected in `snapcarousel` and its corresponding button in `snapcarouselctrl`,
like in the following image when the user drags the scrollbar so as to cause a
change in snap targets from the third image to the fourth image:

![Snap Carousel Explainer](https://i.imgur.com/6BtkRIt.png)

To do this, the developer will need to know when the image the carousel is
snapped to has changed and what it has changed to.

To know when it has changed, they might rely on scroll and scrollend events but
this means their script must run on every scroll. This also limits their
awareness to cases where the change in snap targets occured due to a scroll but
a layout change in the page might also trigger a change in which element the
carousel is snapped to without causing a scroll.

To know what it has changed to, the developer would have to examine the all the
snap areas of the carousel and try to identify which element is snapped to based
on the current layout and style of the page, so they might write something like:

```
// Get the offset, relative to the viewport, at which |area|'s left edge needs
// to be in order to be snap-aligned to |container|.
function GetAlignedOffset(container, area) {
  const areaStyle = getComputedStyle(area);
  const alignment = areaStyle.scrollSnapAlign;
  const scrollMargin = parseInt(areaStyle.scrollMargin);
  const containerOffsetLeft = container.offsetLeft;

  let alignedOffset;
  if (alignment === "start") {
    alignedOffset = containerOffsetLeft;
  } else if (alignment === "center") {
    alignedOffset = containerOffsetLeft + container.clientWidth / 2 - area.clientWidth / 2;
  } else { // "end" alignment
    alignedOffset = containerOffsetLeft + container.clientWidth - area.clientWidth;
  }

  return alignedOffset + scrollMargin;
}

// Get the distance of |area|'s left edge, relative to the viewport from its
// snap-aligned position.
function GetSnapDistance(container, area) {
  const alignedOffset = GetAlignedOffset(container, area);

  return Math.abs(parseInt(area.getBoundingClientRect().x) - alignedOffset);
}

// Update the UI to highlight the item that is snapped-to if it has changed.
function UpdateCurrentSnapTarget() {
  const snapAreas = carousel.querySelectorAll(".slide_thumbnail");
  let newSnapTarget = currentSnapTarget;
  let min_distance = Infinity;

  for (const snapArea of snapAreas) {
    const distance = GetSnapDistance(carousel, snapArea);
    if (distance < min_distance) {
      newSnapTarget = snapArea;
      min_distance = distance;
    }
  }

  DeEmphasize(hintSnapTarget, /*hint*/true);
  if (newSnapTarget !== currentSnapTarget) {
    DeEmphasize(currentSnapTarget);
  }

  currentSnapTarget = newSnapTarget;
  Emphasize(currentSnapTarget);
}

// Update the UI to highlight, as a hint, the item that the current scrolling
// is intending to snap to, unless the item is the last snapped-to thing.
function UpdateHintSnapTarget() {
  const snapAreas = carousel.querySelectorAll(".slide_thumbnail");
  let newHintTarget = hintSnapTarget;
  let min_distance = Infinity;

  for (const snapArea of snapAreas) {
    const distance = GetSnapDistance(carousel, snapArea);
    if (distance < min_distance) {
      newHintTarget = snapArea;
      min_distance = distance;
    }
  }

  if (newHintTarget === hintSnapTarget) {
    return;
  }
  
  DeEmphasize(hintSnapTarget, /*hint*/true);

  // Don't override the current snap target's style with hint styling.
  if (newHintTarget === currentSnapTarget) {
    return;
  }

  hintSnapTarget = newHintTarget;
  Emphasize(hintSnapTarget, /*hint*/true);
}

carousel.addEventListener("scrollend", () => {
  UpdateCurrentSnapTarget();
});

carousel.addEventListener("scroll", () => {
  UpdateHintSnapTarget();
});
```
which is trying to determine which element the browser has selected as the snap
target based on the scroll position but may not align with what the browser
actually picks. [This slideshow example](https://davmila.github.io/SnapEventExamples/carousel-2/proxy.html) uses scroll and scrollend events to synchronize the styles of the images in the carousel and the smaller thumbnails below.
One thing to note in this example is that when clicking on a thumbnail below to scroll
the main carousel, all the thumbnails between the one corresponding to the 
current snap target and the one corresponding to the new one are momentarily highlighted 
as the carousel scrolls to its new snap target. This happens because the `scroll` event 
is not aware of the eventual snap target, causing the page to invoke 
the style change unnecessarily several times. `scrollsnapchanging`
is proposed as an event that lets the page know as early as possible what the
eventual snap target is. Alternatively, an author might use [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API),
but that only goes as far as telling them that a snap area is visible within the
scroll port. So, if multiple snap areas are within the snap port, they would still
need to rely on script similar to the above to determine which snap area is the
snap target.

In cases where multiple elements could be considered snap-aligned, e.g. grid of
snap areas, the developer would need to write a bit more code to figure out
which, among the aligned areas, the user-agent has selected. They would need to
either:
* implement the user agents' algorithm to
  [select between multiple aligned areas](https://drafts.csswg.org/css-scroll-snap/#multiple-aligned-snap-areas), or
* test for which element the user agent has snapped to by shifting layout to see
  which element the snap container follows.

A robust implementation would also need to account for several other details that affect the
user-agent's choice of snap targets, such as:

* [proximity strictness](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-snap-type#proximity) scroll snapping,
* snap areas at the extremes of their snap containers such that their theoretical aligned position cannot be reached,
* snap areas the user-agent considers snapped-to by virtue of their being larger than the scroll port,
* out-of-flow-positined snap areas which may be within the scroll container's subtree in the DOM but not be within the scrollable overflow area of the scroll container,
* scroll-margin; this is accounted for in the sample code above, but any future CSS property which affects scroll-snapping like scroll-margin does could render any existing snap-point calculating code incorrect.

It's also worth noting that user-agents already handle all of this.

## Proposed Solution

We propose new JavaScript events for scroll snap containers
called `scrollsnapchange` and `scrollsnapchanging`. `scrollsnapchange` is
dispatched when a new snap target has been snapped to. `scrollsnapchanging` is
dispatched as soon as the UA has determined a new snap target during an ongoing
scroll operation.

The `scrollsnapchanging` event fires while scrolling is happening,
allowing the page to communicate to the user that their scrolling gesture has
triggered the selection of a new snap point which the user-agent will settle on
when the user's gesture is complete.

The `scrollsnapchange` event fires when scrolling is complete and the snap
point that has been settled on is different from the last snap point that was
settled on.

Both events would be triggered by a layout shift resulting in a change in the
selected snap target.

In the case of the layout example above, the developer would only need to write
the following to detect changes in snap targets:

```
carousel.addEventListener("scrollsnapchange", (evt) => {
  DeEmphasize(hintSnapTarget, /*hint*/true);
  DeEmphasize(currentSnapTarget);
  currentSnapTarget = evt.snapTargetInline;
  Emphasize(currentSnapTarget);
});

carousel.addEventListener("scrollsnapchanging", (evt) => {
  DeEmphasize(hintSnapTarget, /*hint*/true);
  hintSnapTarget = evt.snapTargetInline;
  // Ensure to undo any effects snapchanging away from the
  // currentSnapTarget may have done.
  Emphasize(hintSnapTarget, /*hint*/evt.snapTargetInline !== currentSnapTarget);
});
```
without the risk of computing the wrong thing.
[This slideshow example](https://davmila.github.io/SnapEventExamples/carousel-2/real.html)
is visually identical to the slideshow example mentioned earlier but it uses the
`scrollsnapchanging` and `scrollsnapchange` event listeners above instead of
scroll/scrollend events.*

Additionally [here](https://codepen.io/argyleink/pen/oNOWwKq) is a date-time 
picker example which uses snap events to update the displayed date/time.*

*At the moment, examples using `scrollsnapchanging` and `scrollsnapchange`
are only functional in Chrome with experimental web features enabled.
<br>

## Event Interfaces

**Type**: scrollsnapchange (inspired by [`snapped` comment](https://github.com/w3c/csswg-drafts/issues/156#issuecomment-695085852))  
**Interface**: SnapEvent  
**Sync / Async**: Async  
**Bubbles**: No, except at document  
**Trusted Targets**: Element, Document  
**Cancelable**: No  
**Default action**: None  
**Context (trusted events):** 

<br>

**Type**: scrollsnapchanging (inspired by [`snapped` comment](https://github.com/w3c/csswg-drafts/issues/156#issuecomment-695085852))  
**Interface**: SnapEvent  
**Sync / Async**: Async  
**Bubbles**: No, except at document  
**Trusted Targets**: Element, Document  
**Cancelable**: No  
**Default action**: None  
**Context (trusted events):** 
<br>


These events implement a `SnapEvent` interface.

```
interface SnapEvent : Event {
  readonly attribute EventTarget target;

  readonly attribute Node? snappedTargetBlock;
  readonly attribute Node? snappedTargetInline;
};
```

- `Event.target`: scroll container the snap target is in (this is inherited from the generic DOM [Event](https://dom.spec.whatwg.org/#interface-event) interface).
- `SnapEvent.snappedTargetBlock`: the element which is currently snapped to in the block axis.
- `SnapEvent.snappedTargetInline`: the element which is currently snapped to in the inline axis.

## Alternatives and Other Considerations

The SnapEvent interface described above reflects the minimum amount of information
which satisfies many of the scenarios expressed by developers as can be seen in
the following links:

### Interest in Snap Events

* https://stackoverflow.com/questions/66852102/css-scroll-snap-get-active-item
* https://stackoverflow.com/questions/71007514/css-scroll-snap-focusing-on-the-element-which-got-snapped-to
* https://github.com/w3c/csswg-drafts/issues/7430
* https://stackoverflow.com/questions/75471179/how-to-identify-the-element-currently-visible-on-the-screen-i-am-using-scroll
* https://github.com/tailwindlabs/tailwindcss/discussions/6450
* https://stackoverflow.com/questions/54797620/css-scroll-snap-api
* https://webwewant.fyi/wants/65/
* https://stackoverflow.com/questions/57326493/how-can-i-snap-scroll-and-trigger-the-event-when-snapped?rq=3
* https://stackoverflow.com/questions/53355384/indicators-dots-with-css-scroll-snap

Common to all these examples is the desire to know which element was snapped to,
which the above-described API provides.

### Additional SnapEvent Fields

#### SnapEvent.previousSnapTarget{Block|Inline}

In many use cases, when a change in snap targets occurs, a developer would want
to de-emphasize the element that was previously snapped to and emphasize the
element that is currently snapped to. `previousSnapTarget` would provide a
convenient way to do this by indicating the element that was previously snapped to.
This might be purely a convenience for developers as they could track
this across different occurrences of the snap events. However, in a scenario with an
arbitrary number of snap containers, this convenience could prove to have
substantial value as, without it, the developer would need to track the
currently snapped element for each snap container.

#### SnapEvent.snapAreas{Block|Inline}

Present but less common is the desire to know all the elements which are
considered snap areas, which could be useful for the use case of building a
progress indicator. This is one additional piece of information that could be
added to the SnapEvent interface if there appears to be strong enough desire for
it.

#### SnapEvent.alignedSnapAreas{Block|Inline}

It is possible for multiple snap areas within a snapport to appear visually
snap-aligned, even though user-agents only select one element as they must know
which element to track during layout shifts. For this
case, it might be worth considering whether to expose the set of elements which
are so aligned.

Adopting the above ideas would result in the following SnapEvent interface:

```
interface SnapEvent : Event {
  readonly attribute EventTarget target;

  readonly attribute Node? snapTargetBlock;
  readonly attribute Node? snappTargetInline;

  readonly attribute Node? previousSnapTargetBlock;
  readonly attribute Node? previousSnapTargetInline;

  readonly attribute sequence<Node>? alignedSnapAreasBlock;
  readonly attribute sequence<Node>? alignedSnapAreasInline;

  readonly attribute sequence<Node>? snapAreasBlock;
  readonly attribute sequence<Node>? snapAreasInline;
};
```

### Dispatching SnapEvents to the snap target rather than the snap container

Instead of dispatching the snap events to the snap container, we could dispatch
them to the element that was snapped to. This makes the events
less similar to scroll and scrollend events which are dispatched to the container.

If a page has several snap containers and a developer wanted to style the
snapped-to element according to the container it belonged to, the event would need
to contain information about which scroll container the snapped-to element belongs to and in which axis it
was snapped to. (Although in theory a developer could write some JavaScript to
figure this out, it would seem less than ideal and quite possibly prone to
errors). Catering to this style-per-container scenario would be roughly equivalent to
dispatching the event to the container as proposed and perhaps suggests giving preference to
maintaining similarity with scroll and scrollend events.

Additionally, this might not work naturally for pseudo-elements which can
be snap targets as there is not currently a way to interact with pseudo-elements
via JavaScript.

### One Event vs. Two Events

Much of what can be achieved with `scrollsnapchanging` and `scrollsnapchange` can
probably be achieved with `scrollsnapchanging` and the existing `scrollend` event.
If a developer wants to make certain style adjustments when a `scrollsnapchange`
event occurs, they could simply use `scrollsnapchanging` to track the most recent
choice of snap targets and, upon `scrollend`, perform their style adjustments
according to the tracked information as desired.

One edge case this might not cover is the case where a developer wants to style
`scrollsnapchange` targets differently from `scrollsnapchanging` targets.
Specifically, they would not be able to make this differentiation for a layout
change that would lead to a `scrollsnapchange` event without a scroll occuring
(in which case no `scroll` or `scrollend` event would be dispatched, like in 
[this scroll/scrollend example](https://davmila.github.io/SnapEventExamples/carousel/proxy.html) where you can delete slides. Its snap event counterpart 
is [this example](https://davmila.github.io/SnapEventExamples/carousel/real.html)). 
It might be possible to
bridge this particular gap by having `scroll` events fire before
`scrollsnapchanging` events. By doing so, if a developer encounters a
`scrollsnapchanging` event without having observed a `scroll` event (like in the
layout change scenario described) they would know not to expect a `scrollend`
event and treat that `scrollsnapchanging` event like a `scrollsnapchange` event.

While bridging this gap with `scroll` events seems feasible, it does leave the
developer with a little bit more state to track and more edge cases to handle across JavaScript event listeners. It seems preferable to be able to rely on a dedicated `scrollsnapchange` event.

Additionally, the pattern of changing/changed events could offer some ergonomic
value to developers given such analogous precedents as `scroll`/`scrollend` events,
and [change](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/change_event)/[input](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event) events.

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/1) for comments and questions, or a Pull Request to offer changes 🙏
