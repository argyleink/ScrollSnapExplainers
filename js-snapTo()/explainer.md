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

### Goals

To empower the scroll snapping container with an API for iterating through snap children in a reasonable way, for all snapping containers, including [2-dimensional matrix layouts](https://codepen.io/argyleink/pen/MWWpOmz) and support for all document directions and writing modes.

### Use Cases

Wiring up "snap to next", "snap to last" or "snap to the clicked item" should be trivial methods to call on a snap container:

1. Carousels
2. Sliders
3. Tabousels or Carotabs
4. Media galleries
5. Slides
6. Click/tap and snap to item

<br>

## Proposed Solution

### snapTo(`<axis>`, `<string>` or `<node>`)
- `<axis>` accepts
    - strings (x | y | inline | block | both)
- `<string | node>` accepts
    - strings (next | prev | first | last)
    - node
        - must be a child of the scroller and a registered snap child of the snapport

Example usage:  
```js
scrollSnapContainer.snapTo('x', 'next')
scrollSnapContainer.snapTo('y', 'prev')
scrollSnapContainer.snapTo('inline', 'first')
scrollSnapContainer.snapTo('block', 'last')
scrollSnapContainer.snapTo('x', scrollSnapContainer.querySelector('.child-2'))
```

<br>

### Edge Cases
#### `"next"` is requested when at the end
If current snapped element is `node.lastElementChild`, then no snapping occurs. The "next" element is the current element, yielding no change in snap position.

#### `"prev"` is requested when at the end
If current snapped element is `node.firstElementChild`, then no snapping occurs. The "prev" element is the current element, yielding no change in snap position.


<br>

## Examples
### Example 1 `.snapTo('x', 'next')`
The browser should scroll in that direction the same amount as if the `right arrow key` was pressed and perform it's routine regarding proximity and mandatory next snap target discovery.

### Example 2 `.snapTo('inline', 'next')`
If the browser is set to `rtl`, then the browser should scroll left the same amount as if the `left arrow key` was pressed and perform it's routine regarding proximity and mandatory next snap target discovery.

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/6) for comments and questions, or a Pull Request to offer changes 🙏
