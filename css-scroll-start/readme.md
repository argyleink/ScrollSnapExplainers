# CSS `scroll-start`

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to
problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the
most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

Scroll experiences don't always start at the beginning. Interactions with carousels, swipe controls, and listviews often start somewhere in the middle, and each require Javascript to set this position on page load. By enabling CSS to specify this scroll start x or y position, both users, page authors and browsers benefit:
- Colocate initial scroll position with other scroll styles
- JavaScript can be removed
- Post page load scroll shift mitigated

### Goals
Reduce Javascript responsibility and enable a declarative pattern for interaction models that need to start somewhere other than the beginning.

### Use Cases
- Carousels
- Pull to refresh
- Pull to search
- ListViews starting at a certain item or group
- Galleries
- Swipe interactions (left to archive, right to delete)

<br>

## Proposed Solution
A new CSS property on scroll containers `scroll-start` which sets the initial scroll position. Once the user has interacted with the scroll area, `scroll-start` has no effect. The style must be present during scrollport creation or it has otherwised missed it's timing.

|   |   |
|:----------|:-------------| 
| Name: | `scroll-start` || `scroll-start-x` `scroll-start-y` `scroll-start-inline` `scroll-start-block` |  
| Value: | `auto` `<length-percentage>` [`<element>`](https://drafts.csswg.org/selectors-4/#typedef-id-selector) |  

Features:
- Allows setting an absolute scroll value on either axis
- Allows setting both axes start positions at the same time
- Allows setting an element as the target for an axis

<br>

### Interactions with [fragment navigation](https://html.spec.whatwg.org/multipage/browsing-the-web.html#scroll-to-fragid)
If the scrollport has a in-page `:target` via a URL fragment or a previous scroll position, then `scroll-start` is unused. Existing target logic should go unchanged. This makes `scroll-start` a soft request in the scroll position resolution routines. 

<br>

## Examples
#### 1. Set the start position to a snap child
```css
.snap-scroll-inline {
  overflow-inline: scroll;
  scroll-snap-type: inline mandatory;
  scroll-start-inline: selector(#snap-scroll-inline-hero);
}
```

#### 2. Set the start position to an absolute value
```css
:root { --nav-height: 100px }

.snap-scroll-y {
  scroll-start-y: var(--nav-height);
}
```

#### 3. Set both with a shorthand (inline then block)
```css
.scroll {
  scroll-start: 200px 400px;
}
```

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/4) for comments and questions, or a Pull Request to offer changes 🙏
