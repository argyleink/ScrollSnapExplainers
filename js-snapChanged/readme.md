# JS snapChanged event

## Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to
problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the
most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

todo

## Goals

* todo

## Use Cases

- Scroll centric selection interfaces, like a dial or carousel
- Should only fire if user interaction has ended, and a new item has been truly rested on. If a user is still touching the screen or the touchpad, this event should not fire, even if the scroll position is exactly at a snapped elements position. 

## Proposed Solution

todo

### Example 1: todo

todo

```js
scrollingSnapContainer.addEventListener('snapchanged', event => {
  console.info(event)
  // todo
})
```

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/1) for comments and questions, or a Pull Request to offer changes 🙏
