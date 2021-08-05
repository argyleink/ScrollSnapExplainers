# JS `snapchanging` event

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

CSS scroll snap points are often used as a mechanism to create scroll interactive "selection" components, where selection is determined with javascript with intersection observers and a scroll end guestimate. By creating a built-in event, the invisible state will become actionable, at the right time, and always correct.

## Goals

Help developers sychronize a snapped scroll item with the rest of their interface elements efficiently and effectively.

Should only fire if user interaction has ended, and a new item has been truly rested on. If a user is still touching the screen or the touchpad, this event should not fire, even if the scroll position is exactly at a snapped elements position. 

## Use Cases

Scroll centric selection:
- Date / Time picker
- Carousel
- Tabs
- State / Country picker

<br>

## Proposed Solution

A new javascript event for scroll snap container elements called `snapchanging`

<br>

## Examples

#### 1: Carousel is about to change selection

```js
carouselSnapContainer.addEventListener('snapchanging', event => {
  console.info(event)
})
```

#### 2: Tabs have begun snapping to a new element

```js
tabsSnapContainer.onsnapchanging = event = > {
  console.info(event)
}
```

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/1) for comments and questions, or a Pull Request to offer changes 🙏
