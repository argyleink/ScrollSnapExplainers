# JS `snapchanged` event

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

CSS scroll snap points are often used as a mechanism to create scroll interactive "selection" components, where selection is determined with javascript intersection observers and a scroll end guestimate. By creating a built-in event, the invisible state will become actionable, at the right time, and always correct.

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

## Proposed Solution

A new event for scroll snap containers called `snapchanged`. The event is dispatched when a new snap target has been snapped to.

- Should dispatch if user scroll interaction has ended and a new item has been rested on. If a user is still touching the screen or the touchpad, this event should not fire, even if the scroll position is exactly at a snapped elements position. 
- Should dispatch if animations or transitions change the snapped style of the container or children, IF they have in fact changed the snap target.

<br>

**Type**: snapchanged (inspired by [`snapped` comment](https://github.com/w3c/csswg-drafts/issues/156#issuecomment-695085852))  
**Interface**: SnapEvent  
**Sync / Async**: Async  
**Bubbles**: No  
**Trusted Targets**: Element, Document  
**Cancelable**: No  
**Composed**: Yes  
**Default action**: None  
**Context (trusted events):** 

<br>

- `Event.target`: scroll container the event target is in
- `SnapEvent.snappedTargetBlock`: element which has been newly snapped to in the block axis.
- `SnapEvent.snappedTargetInline`: element which has been newly snapped to in the inline axis.
<br>

```
interface SnapEvent {
  readonly attribute EventTarget target;

  readonly attribute Node? snappedTargetBlock;
  readonly attribute Node? snappedTargetInline;
};
```

<br>

## Examples

#### 1: Carousel has snapped to a new element

```js
carouselSnapContainer.addEventListener('snapchanged', event => {
  console.info(`did snap to ${event.snapTargetBlock.id}`);
})
```

#### 2: Tabs have snapped to a new element

```js
tabsSnapContainer.onsnapchanged = event = > {
  console.info(`did snap to ${event.snapTargetInline.id}`);
}
```

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/1) for comments and questions, or a Pull Request to offer changes 🙏
