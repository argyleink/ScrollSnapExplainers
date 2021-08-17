# CSS `::scroll-marker`

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to
problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the
most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction
Scroll is found everywhere, from carousels to horizontal product grids that flow offscreen to virtual scrollers and cyclical scrollers. A common user experience and ask is to get rid of the "bar" that shows the scroll, and replace it with another kind of scroll indicator (like dots). Scrolling is not always tied to a bar, and combined with snap points, a marker feature is especially useful.

![Scroll bar vs. scroll markers](https://developer-chrome-com.imgix.net/image/HodOHWjMnbNw56hvNASHWSgZyAf2/0gNU49FcXulOlQ3q5Voe.png)

### Goals
Reduce Javascript responsibility and enable a declarative pattern for styling scroll/snap points that doesn't rely on a bar.

### Use Cases
- Carousels (hero carousel, iTunes-inspired album flow carousels)
- Product lists (i.e. App store-like experiences)
- Scrollable lists with corresponding markers
- Galleries

<br>

## Proposed Solution
A new CSS property on scroll containers named `scroll-display` which converts the scroll bar area into a marker area, paired with a new CSS pseudo element `::scroll-marker` that enables styling of said markers in a similar manner to list `::marker`. This means that you could create custom counter styles, and adjust the spacing of the items.

|   |   |
|:----------|:-------------| 
| Name: | `scroll-display` |
| Values: | `auto` `marker` `bar` |
| Default: | `bar` |

<br>

```css
.carousel {
  scroll-display: marker; /* set scroll display type */
}

.carousel::scroll-marker {
  color: yellow;
}

.carousel::scroll-marker:focus {
  color: red;
}
```

<br>

**Features:**
- Allows setting marker-like styles instead of bar style for scroll
- Allows styling markers
- Enables positioning markers within the scroll area


<br>

## Additional Thoughts / Open Questions

### How to set/determine scroll points

List item = scroll child for the sake of this conversation, but a scroll child could be any element:

- Scroll points could be set one of two ways:
    - Set on the parent scroller for each list item (dynamic scroll points)
    - Manually set on each list item by the author (scrollable attribute)
- Keeping correct state for the active list item and list marker
- Can we also include previous/next buttons, like oldschool scrollbars had at the top and bottom?

<br>

## Examples

TBD

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/16) for comments and questions, or a Pull Request to offer changes 🙏
