# JS `snapTo()`

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to
problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the
most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [W3C](https://www.w3.org)
* Current version: this document

## Introduction

[Scroll snap points](https://www.w3.org/TR/css-scroll-snap-1/) are an important and powerful CSS feature. They enable the creation of pointer agnostic stepped scroll experiences, and do so very succinctly. The problem is, they're often used alongside adjacent elements which need to represent this stepped state of scroll, aka the user needs a way to see the state of the snap. Developers today write Javascript observers or create custom functions to monitor and drive this state. 

Wiring up "snap to next", "snap to last" or "snap to the clicked item" should be trivial methods to call on a snap container.

### Goals

To empower the scroll snapping container with an API for iterating through snap children in a reasonable way, for all snapping containers, including [2-dimensional matrix layouts](https://codepen.io/argyleink/pen/MWWpOmz) and support for all document directions and writing modes. 

### Use Cases

1. Carousels
2. Sliders
3. Tabousels or Carotabs
4. Media galleries
5. Slides
6. Click/tap and snap to item

<br>

## Proposed Additions
New values for the [`scrollToOptions`](https://developer.mozilla.org/en-US/docs/Web/API/ScrollToOptions) dictionary of the [CSSOM View](https://drafts.csswg.org/cssom-view/#dictdef-scrolltooptions). This would enable APIs like `Window.scrollTo()`, `Element.scrollBy()` and `Element.scrollIntoView()` to be aware and offer APIs to programmatic snapping.

**New values for scrolling containers:**
- `snap-next`
- `snap-prev`
- `snap-first`
- `snap-last`

**New values for scroll children:**
- `snap-self`

<br>

## Examples

#### 1: basic
```js
container.scrollTo({
  left: 'snap-next',
})
```

#### 2: Control animation
```js
container.scrollTo({
  left: 'snap-prev',
  behavior: 'smooth',
})
```

#### 3: 2D
```js
container.scrollTo({
  left: 'snap-first',
  top: 'snap-first',
  behavior: 'instant',
})
```

#### 4: To a specific element
```js
container.querySelector('.child-2').scrollIntoView({
  top: 'snap-self',
  behavior: 'smooth',
})
```

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/13) for comments and questions, or a Pull Request to offer changes 🙏
