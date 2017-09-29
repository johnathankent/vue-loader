# Scoped CSS

When a `<style>` tag has the `scoped` attribute, Vue-loader prevents styles from applying to the global space by using PostCSS to automatically add a data-attribute to all the component's elements. This method ensures it's CSS will cascade only to elements of the current component and it's children. It does not require polyfills.

To provide the ability to let parent components control layout of children components, the root element of child components also receives the data-attribute.

> _**Important note**: Because style cascading inheritance rules apply to the computed DOM, styles (including scoped) may cascade to elements outside the Vue template file. Make sure to check out the [styleguide](#styleguide)._

``` html
<!-- Vue component template -->
<style scoped>
.example {
  color: red;
}
</style>

<template>
  <div class="example">hi</div>
</template>
```

``` html
<!-- rendered.html -->
<style>
.example[data-v-f3f3eg9] {
  color: red;
}
</style>

<div class="example" data-v-f3f3eg9>hi</div>
```

## Mixing Local and Global Styles

You can include both scoped and non-scoped styles in the same component:

``` html
<style>
/* Global styles */
</style>

<style scoped>
/* These cascade only to children elements of this component */
</style>
```

## Layout and Child Components

Setting `scope` on a parent component style will mean that a child component's root node will also receive a scoping data-attribute. This is an intentional design feature so that the the parent can style the child root element's layout.

In this example of a scoped parent and scoped child, the rendered child component's HTML has both `data-v-f3f3eg9` and `data-v-gd78ef7` attributes:

```html
<!-- rendered.html -->
<div class="container" data-v-f3f3eg9>
  <div class="parent" data-v-f3f3eg9>Parent</div>
  <div class="child container" data-v-f3f3eg9 data-v-gd78ef7>Child</div>
</div>
```

> _**Important Note**: A child component's root node can be affected by both parent's scoped CSS (or inherited rules) and child's scoped CSS. Here is a playground where you can see how CSS **specificity** and **inheritance** cascade with the above rendered HTML: https://codepen.io/anon/pen/yzgVdx_

## Dynamically Generated Content

Data-scoping attributes are not applied to DOM content created with `v-html`. You can style dynamically generated content using [deep selectors](#deep-selectors).

## Advanced CSS styling

<h3 id="deep-selectors">Deep Selectors</h3>

If you want a selector in `scoped` styles to be "deep", i.e. directly affecting child components, you can use the `>>>` combinator:

``` html
<style scoped>
.a >>> .b { /* ... */ }
</style>
```

The above will be compiled into:

``` css
  .a[data-v-f3f3eg9] .b { /* ... */ }
```

Some pre-processors, such as SASS, may not be able to parse `>>>` properly. In those cases you can use the `/deep/` combinator instead - it's an alias for `>>>` and works exactly the same.

### Custom Shadow DOM Elements (true style encapsulation)

True encapsulation where styles do not leak in or out require custom web component elements which use Shadow DOM (not yet fully supported by browsers). Here is an example of how to add custom elements to your Vue application: https://github.com/karol-f/vue-custom-element.

> **Note:** Custom web component elements are not yet widely adopted by browsers. Plan on required polyfils for older browsers, longer devlepment times, and render performance hits if you plan on implementing it in your project.

<h1 id="styleguide">Styleguide</h1>

It can be difficult to avoid style conflict with so many devices, frequently changing browsers, responsive requirements, and with reactive applications -- dynamic changes to DOM.

Using vue-loader's [Scoped CSS](/features/scoped-css.md), CSS sourcemaps, browser inspector tools, and frameworks aid development, but conflicts still happen.

## Scoped styles do not eliminate the need for classes.
  Due to the way browsers render various CSS selectors, `p { color: red }` will be many times slower when scoped (i.e. when combined with an attribute selector). If you use classes or ids instead, such as in `.example { color: red }`, then you virtually eliminate that performance hit. [Here's a playground](http://stevesouders.com/efws/css-selectors/csscreate.php) where you can test the differences yourself.

## Be careful with descendant selectors in recursive components!
For a CSS rule with the selector `.a .b`, if the element that matches `.a` contains a recursive child component, then all `.b` in that child component will be matched by the rule.

## Troubleshooting

### Understand CSS Cascading inheritance and specificity.

Styles are inherited and cascade as applied to the computed DOM and should not be confused with applying only to the Vue template file.

> **Troubleshooting tip**: To determine the conflicting cascade **inheritance** and **specificity**, examine the compiled style rules on the computed DOM using browser inspector developer tools.

- [Read about Cascade and Inheritance from Mozilla](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Cascade_and_inheritance)
- [Read about CSS Inheritance from Sitepoint](https://www.sitepoint.com/css-inheritance-introduction)

### Know where conflicts come from

Since CSS is always rendered to the computed DOM, it is wise to be aware of all external factors in your Vue project:

- browser defaults
- user or client added styles _(remember this when troubleshooting)_
- external files:
    - linked from `<head>`
    - CSS @imports
    - themes or frameworks
- inline `script` attributes on HTML elements
- dynamically added via JavaScript
- defaults from custom elements or node packages
- component `.vue` files:
    - linked to using `<style src="[external styles file]"></style>`
    - `<style>`
    - `<style scoped>`
    - dynamically added from `<script>`
    - found in `<template>`
- custom web components (including elements using shadow DOM)

## Consider using style patterns

### A simple wrapper

Style conflicts from parent to nested child can occur _with or without_ the parent component adding data-scoping attribute to the child root node. Therefore, it may be easier to always assume that parent's scoped data-scoping attribute may be applied at anytime.

Since data-scoping attribute makes the CSS more specific, overriding style conflicts requires _even more_ specificity. A simple method to avoid common style conflicts on the component root node, is to use a wrapper node where the parent can apply style:

``` html
<!-- Vue component template -->
<template>
  <div> <!-- <<== Assume scoped parents will use this for layout. -->
    <div class="example">hi</div> <!-- <<== Use all nested nodes for selectors. -->
  </div>
</template>
```

### Using selectors to communicate a pattern
``` html
<!-- Vue component template: FOO.vue -->
<template>
  <div>
    <div class="example">hi</div>
    <child-component id="child" class="layout-from-FOO"></child-component> <!-- <<== Only apply layout here. -->
  </div>
</template>

<style>
  /* global styles */
  .layout-from-FOO { /* use for layout only, applies to all rendered elements with this class */ }
</style>

<style scoped>
  /* These cascade only to to children elements of this component */
  .layout-from-FOO { /* use for layout only; applies only to this child component. */ }
</style>
```

### Example patterns for project styleguides

#### For large projects or teams, dynamic templates, most developers:

```markdown
# When adding styles to Vue Component Templates:
Prevent conficting rules of parent and child components to avoid CSS selector specificity or inheritance conflicts:

### DO:
- Wrap common classes with `<style scoped>`.
- Put global styles that affect all components in `[layout|theme|global|variables].css`

### DO NOT:
- Apply styles to elements without ids or classes. (It's a performance hit.)
- Use "Deep Selectors". Put them in the global theme or variable stylesheets.
- Style the root node; leave it for the parent to control layout.
- Style child components.
    - **Exception:** Layout styles using non-common classnames and resolve only to child component's root node.
```

#### For small projects or teams, or CSS-gurus:

```markdown
# When adding styles to Vue Component Templates:
Prevent conficting rules of parent and child components to avoid CSS selector specificity or inheritance conflicts:

### DO:
- Wrap common classnames with `<style scoped>`.
- Put global styles that affect all components in `[layout|theme|global|variables].css`

### Avoid:
- Using unnecessary "Deep Selectors". Put them in global theme or variable stylesheets.
- Styling the root node; leave it for the parent to control layout.
- Styling child components.
    - **Exception:** Layout styles that resolve only to child component's root node.
```
