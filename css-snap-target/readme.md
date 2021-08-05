# CSS Snapped Elements pseudo-classes
> `:snapped-x-target` `:snapped-y-target` `:snapped-inline-target` `:snapped-block-target`

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

User Agents privately maintain scroll snap state, leaving web developers unable to update their interfaces accordingly. 

A scroll snap container can have any nested child snapped to either the x or y axes (or inline/block respectively). Since its 2-dimensional, a UA is likely unable to report a single snapped element for a container which is what most authors will expect. By restricting the pseudo-class to an axis, the state request is more specific and easier to fulfill.

### Goals

Reduce Javascript responsibility and enable a declarative pattern for updating interface children if they are currently snapped.

### Use Cases

- Carousels
- ListViews (great for making selections in long lists)
- Galleries (almost always an active item with additional UI to show)

<br>

## Proposed Solution

```css
:snapped-inline-target {
  ...
}

section:snapped-block-target > header {
  ...
}
```

<br>

## Examples

#### 1: Lifted card snap child

```css
.card:snapped-inline-target {
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
