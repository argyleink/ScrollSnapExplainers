# CSS Snapped Elements pseudo-classes
> `:snapped` `:snapped-x` `:snapped-y` `:snapped-inline` `:snapped-block`

### Status of this Document
This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Active**
* Expected venue: [CSSWG](https://drafts.csswg.org/)
* Current version: this document

## Introduction

User Agents privately maintain scroll snap state, leaving web developers unable to update complimentary interfaces accordingly. Scroll snapping is often used for "picker UX", [made popular by iOS](https://miro.medium.com/max/700/1*LJujvSAe2lGVdY2ETmg5ew.png).  

![Screen Shot 2021-09-13 at 1 09 52 PM](https://user-images.githubusercontent.com/1134620/133149663-b3cd3f6e-7541-4cc0-a325-7fe7ade2e4cb.png)

A scroll snap container can have any nested child (**just one**) snapped to either the x or y axes (or inline/block respectively). In a 2-dimensional snapping layout, a UA may match 2 separate elements with `:snapped`. Adding axes to the pseudo-class makes the selector more specific, as to match design or developer intent. 

In the case where more than 1 element appear snapped or are snapped on an axis, the 1st of the DOM order is the `:snapped` child. An exception is if a child is snapped on the cross axis, in which case it should be `:snapped` even if it's not the 1st of the DOM order.

The selector matching timing should synchronize with the same frame the UA has decided on a new snap child. Sooner is better than later, but UA's can decide.

> A [gallery of scroll snap interactions](https://snap-gallery.netlify.app/) can help illuminate the CSS UI need, here's [the source](https://github.com/argyleink/snapping-gallery). Specifically see the `:snapped` demo's for an attempt at deriving the snapped element.

### Preventing infinite or expensive browser loops
Like `:hover`, a developer may find they have caused a loop by changing a property on the snapped item that causes it to unsnap, then potentially snap again. This type of behavior is a concern mentioned by the CSSWG in [selectors that depend on layout](https://wiki.csswg.org/faq#selectors-that-depend-on-layout) and is top of mind here in this feature. 

This can be mitigated by limiting the loops to a single frame, just like hover. This avoids "tight loops" that could cause major browser problems, it instead is a loose loop which hover has proved for 20 years can be mitigated and avoided through diligence. It's understood there's a challenge here, and the enablement via CSS will pacify all the much worse JavaScript solutions folks attempt today. Hover is a cherished and respected property that has proven to not be as destructive than anticipated.

Worth considering too, only evaluating selectors with `:snapped` if the containing scrollport is actually scrolling. If no scrolling is happening, the selectors won't continue firing. Consider it like the `:hover` hit test, rather a scroll test. The user, or programmatic scroll from JS, are events that will trigger the selector evaluation, not a constant frame by frame evaluation.

### Common Gotcha
Depending on the scroll snap styles, some (many) snap children may never become snapped. This is a recurring UX and API question, as a scroll gesture may be limited or at the end, and the desired snap target cannot be brought to the snap axis alignment area. 

Take the following example, where figure elements want to align to `start` but the container is scrolled to the end. Essentially, the last 2 figures will never be snapped:

![Screen Shot 2021-08-26 at 2 29 18 PM](https://user-images.githubusercontent.com/1134620/131038651-8adfd69d-806b-4915-ba6e-827cde2054f5.png)

**Unless!**  
Styles are added so the user can scroll to the last item, aka enough padding in the container so items at the end can align on the axis:

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
| Name: | `:snapped` or `:snapped-block` `:snapped-y` `:snapped-inline` `:snapped-x` |   

<br>

The loose example, estimated to be the most commonly used, will match all scroll snap targets, regardless of axis. Assuming folks aren't overlapping scroll snap children, this should be quite reliable.

```css
:snapped {
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
.card:snapped {
  --shadow-distance: 30px; /* raised size */
}
```

### 2: Accessible outline on snap child

```css
:snapped-inline {
  outline: 3px solid hotpink;
}
```

### 3: Lifted sticky section header navigation

```css
section:snapped-y > header {
  box-shadow: 0 .5em 1em .5em lch(5% 5% 200);
  border-inline-start-color: hotpink;
}
```

<br>

## Cyclical concerns, like with `:hover`

We ([Robert Flack](https://github.com/flackr), [Tab Atkins](https://github.com/tabatkins) and [Adam Argyle](https://github.com/argyleink/)) believe the value of adding this feature exceeds the implementation difficulties. The lengths at which people go to try and derive the snapped child are never 100% effective and require complex JavaScript algorithms.

We are OK letting `:snapped` cycle like `:hover`, as long as it's only once per lifecycle update. We believe developers will be responsible with this, as they have with `:hover`.

<br>

## Privacy and Security Considerations

### Privacy

There are no known privacy impact of this feature.

### Security

There are no known security impacts of this feature.

## Contributing
For now, please use this [Discussion](https://github.com/argyleink/ScrollSnapExplainers/discussions/5) for comments and questions, or a Pull Request to offer changes 🙏
