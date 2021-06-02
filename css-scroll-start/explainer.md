# CSS scroll-start property

## Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to
problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the
most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

Scroll experiences don't always start at the beginning! Interactions with carousels, swipe controls, and listviews often start somewhere in the middle, and each require Javascript to set this position on page load. By enabling CSS to specify this scroll start x or y position, both users, page authors and browsers benefit:
- Colocate initial scroll position with other scroll styles
- JavaScript can be removed
- Post page load scroll shift mitigated

## Goals

* todo

## Use Cases
- Carousels
- Pull to refresh
- ListViews
- Galleries
- Swipe interactions (left to archive, right to delete)

## Proposed Solution

Spec should:
- Allow setting an absolute scroll value on either axis
- Allow setting both axes start positions at the same time
- Allow setting an element as the target

### Example 1: todo
#### Set the start position to a snap child
```css
.snap-scroll-x {
  overflow: scroll;
  scroll-snap-type: x mandatory;
  scroll-snap-start-x: selector(.snap-scroll-x .child-2);
}
```

#### Set the start position to an absolute value
```css
:root { --nav-height: 100px }

.snap-scroll-y {
  scroll-snap-start-y: var(--nav-height);
}
```

#### Set both with a shorthand
x then y
```css
.snap-scroll {
  scroll-snap-start: 200px 400px;
}
```

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/4) for comments and questions, or a Pull Request to offer changes 🙏
