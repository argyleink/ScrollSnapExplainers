# CSS Snapped Elements pseudo-classes
> `:snapped-target` `:snapped-x-target` `:snapped-y-target` `:snapped-inline-target` `:snapped-block-target`

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

User Agents privately maintain scroll snap state, leaving web developers unable to update their interfaces accordingly. The proposed `snapped-*` pseudo-classes seek to be close to [`:target`](https://www.w3.org/TR/selectors-4/#the-target-pseudo) fragment navigation syntax.

A scroll snap container can have any nested child snapped to either the x or y axes (or inline/block respectively). Since its 2-dimensional, a UA is likely unable to report a single snapped element for a container which is what most authors will expect. Adding axes to the pseudo-class makes the request more specific and able to fulfill 2D scenarios. It will be possible for multiple elements to match this selector.

### Common Gotcha
Depending on the scroll snap styles, some (many) snap children may never become snapped. This is a recurring UX and API question, as a scroll gesture may be limited or at the end, and the desired snap target cannot be brought to the snap axis alignment area. 

Take the following example, where figure elements want to align to `start` but the container is scrolled to the end. Essentially, the last 2 figures will never be snap targets:

![Screen Shot 2021-08-26 at 2 29 18 PM](https://user-images.githubusercontent.com/1134620/131038651-8adfd69d-806b-4915-ba6e-827cde2054f5.png)

Unless!  
Styles are added so the user can scroll to the last item, aka enough padding in the container so items at the end can become snap targets:

![Screen Shot 2021-08-26 at 2 32 49 PM](https://user-images.githubusercontent.com/1134620/131039093-38946e38-a664-4fa0-a9da-f0222cf7b423.png)

### Goals
Reduce Javascript responsibility and enable a declarative pattern for updating interface children if they are currently snapped.

### Use Cases
- Carousels
- ListViews (great for making selections in long lists)
- Galleries (almost always an active item with additional UI to show)

<br>

## Proposed Solution
5 new CSS pseudo-class selectors. 4 for higher specificity targetting, 1 for loose matching:

|   |   |
|:----------|:-------------| 
| Name: | `:snapped-target` or `:snapped-block-target` `:snapped-y-target` `:snapped-inline-target` `:snapped-x-target` |   

<br>

The loose example, estimated to be the most commonly used, will match all scroll snap targets, regardless of axis. Assuming folks aren't overlapping scroll snap children, this should be quite reliable.

```css
:snapped-target {
  outline: hotpink;
}
```

**Features:**  
- Allows loose targeting shorthand for authors with distinct axis snap targets
- Allows specific axis selecting in 2D cases

<br>

## Examples

#### 1: Lifted card snap child

```css
.card:snapped-target {
  --shadow-distance: 30px; /* raised size */
}
```

### 2: Accessible outline on snap child

```css
:snapped-x-target {
  outline: 3px solid hotpink;
}
```

### 3: Lifted sticky section header navigation

```css
section:snapped-y-target > header {
  box-shadow: 0 .5em 1em .5em lch(5% 5% 200);
  border-inline-start-color: hotpink;
}
```

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/5) for comments and questions, or a Pull Request to offer changes 🙏
