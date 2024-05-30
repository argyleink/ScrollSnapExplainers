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
where snapcarousel contains a set of images which can be viewed by
scrolling horizontally and snapcarouselctrl gives an alternative way to scroll
snapcarousel using buttons which can be used to select the image to be centered
within snapcarousel.

A developer might want to synchronize the styles of both the
image selected in snapcarousel and its corresponding button in snapcarouselctrl,
like in the following image when the user drags their finger so as to cause a
change in snap targets from the third image to the fourth image:

![Snap Carousel Explainer](https://i.imgur.com/6BtkRIt.png)

To do this, the developer will need to know when the image the carousel is
snapped to has changed and what it has changed to.

To know when it has changed, they might rely on a scrollend event, but this
limits their awareness to cases where the change in snap targets occured due to
a scroll. However, a layout change in the page might also trigger a
change in which element the carousel is snapped to without causing a scroll.

To know what it has changed to, the developer would have to examine the all the
snap areas of the carousel and try to identify which element is snapped to based
on the current layout and style of the page, so they might write something like:

```
// Determine if a snap container |container| is aligned to an element |target|
// in the x axis.
function isSnapAligned(container, target) {
  const containerBoundingRect = container.getBoundingClientRect();
  const scrollLeft = container.scrollLeft;

  const targetStyle = getComputedStyle(target);
  const alignment = target.style.scrollSnapAlign;
  const targetBoundingRect = target.getBoundingClientRect();
  const scrollMargin = parseInt(targetStyle.scrollMarginLeft);

  let alignedOffset;
  if (alignment === "start") {
    alignedOffset = boundingRect.x - scrollMargin;
  } else if (alignment === "center") {
    alignedOffset = boundingRect.x + boundingRect.width / 2
      - containerBoundingRect.width / 2 - scrollMargin;
  } else { // end alignment
    alignedOffset = boundingRect.x + boundingRect.width
      - containerBoundingRect.width - scrollMargin;
  }

  // Other cases a developer's code must handle include
  // - where the snap position doesn't exactly match the aligned position due
  //   to being at the extremes of the scroll container
  // - cases where the browser has selected a snap target based on its
  //   being larger than the scrollport rather being aligned to the scroll
  //   position.
  return scrollLeft === alignedOffset;
}

// Get the element which a snap container |container| is aligned to.
function getSnappedTarget(container) {
  // In more complicated layouts, we'd need to filter out elements which are in
  // the subtree of |container| but are also within the subtree of a scroll
  // container descendant of |container|.
  const snapAreas = container.querySelectorAll(".snaparea");

  for (const area of snapAreas) {
    if (isSnapAligned(container, area)) {
      // Ignores the possibility of multiple areas being aligned.
      return target;
    }
  }
}

const container = document.getElementById("snapcarousel");

container.addEventListener("scrollend", () => {
  const snapTarget = getSnappedTarget(container);

  HighlightElementAndCorrespondingButton(snapTarget);
});

container.addEventListener("scroll", () => {
  const snapTarget = getSnappedTarget(container);

  HighlightElementAndCorrespondingButtonAsHint(snapTarget);
});


```
which is trying to determine which element the browser has selected as the snap
target based on the scroll position but may not align with what the browser
actually picks.

In cases where multiple elements could be considered snap-aligned, e.g. grid of
snap areas, the developer would need to write a bit more code to figure out
which, among the aligned areas, the browser has selected. They would need to
either:
* implement the user agents' algorithm to
  [select between multiple aligned areas](https://drafts.csswg.org/css-scroll-snap/#multiple-aligned-snap-areas), or
* test for which element the user agent has snapped to by shifting layout to see
  which element the snap container follows.

The proposed `scrollsnapchanging` event fires while scrolling is happening
if the user-agent detects that the scrolling gesture will lead to a new snap
target, e.g. touch scrolling the carousel above to peek at the next image before
deciding to drag it fully to the carousel's center.
To accomplish this, the developer would need to have the above code run on every
scroll event and take action if it detects that the scroll is far enough to
warrant a change in snap targets.

This represents unnecessary script the page must run since the user-agent is
already internally making all of these decisions but not exposing them to the
developer.

## Proposed Solution

New events for scroll snap containers called `scrollsnapchange` and
`scrollsnapchanging`. `scrollsnapchange` is dispatched when a new snap target
has been snapped to. `scrollsnapchanging` is dispatched as soon as the UA has
determined a new snap target during an ongoing scroll operation.
`scrollsnapchanging` allows the developer give the user a hint that their
gesture will result in settling on a new item, different from where they were
previously settled.

In the case of the layout example above, the developer would only need to write
the following to detect changes in snap targets:

```
const snapcarousel = document.getElementById("snapcarousel");

snapcarousel.addEventListener("scrollsnapchange", (evt) => {
  HighlightElementAndCorrespondingButton(evt.snapTargetInline);
});

snapcarousel.addEventListener("scrollsnapchanging", (evt) => {
  HighlightElementAndCorrespondingButtonAsHint(evt.snapTargetInline);
});
```
without the risk of computing the wrong thing.

<br>

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

These events implement a SnapEvent interface.

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
element that is currently snapped to. previousSnapTarget would provide a
convenient way to do this indicating the element that was previously snapped to.
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

It's possible for multiple snap areas within a snapport to appear visually
snap-aligned, even though user-agent only selected one element as they must know
which elements to track during layout shifts. For this
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
them to element that was snapped to. This makes the events
less similar scroll and scrollend events which are dispatched to the container.

If a page has several snap containers and a developer wanted to style the
snapped-to element according to the container it belonged to, the event would need
to contain information about which scroll container the snapped-to element belongs to and in which axis it
was snapped to. (Although in theory a developer could write some JavaScript to
figure this out, it would seem less than ideal and quite possibly prone to
errors). This style-per-container scenario would be roughly equivalent to
dispatching the event to the container as described above and perhaps suggests giving preference to
maintaining similarity with scroll and scrollend events.

### One Event VS Two Events

Much of what can be achieved with `scrollsnapchanging` and `scrollsnapchanged` can
probably be achieved with `scrollsnapchanging` and the existing `scrollend` event.
If a developer wants to make certain style adjustments when a `scrollsnapchanged`
event occurs, they could simply use `scrollsnapchanging` to track the most recent
choice of snap targets and, upon `scrollend`, perform their style adjustments
according to the tracked information as desired.

One edge case this may not cover is the case where a developer wants to style
`scrollsnapchanged` targets differently from `scrollsnapchanging` targets.
Specifically, they would not be able to make this differentiation for a layout
change that would lead to a `scrollsnapchanged` event without a scroll occuring
(in which case no `scroll` or `scrollend` event would be dispatched). It might be possible to
bridge this particular gap by having `scroll` events fire before
`scrollsnapchanging` events. By doing so, if a developer encounters a
`scrollsnapchanging` event without having observed a `scroll` event (like in the
layout change scenario described) they would know not to expect a `scrollend`
event and treat that `scrollsnapchanging` event like a `scrollsnapchanged` event.

While bridging this gap with `scroll` events seems feasible, it does leave the
developer with a little bit more work to do and leaves their application
subject to a little bit more uncertainty tied to whether the correct events
fired were fired and handled in the correct order whereas there might be less uncertainty
and less work to do if they could rely on a dedicated `scrollsnapchanged` event.

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
