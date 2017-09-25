# Scoped CSS

When a `<style>` tag has the `scoped` attribute, Vue-loader prevents styles from applying to the global space by using PostCSS to automatically add a data-attribute to all the component's elements. This method ensures it's CSS will cascade only to elements of the current component and it's children. It does not require polyfills.

To provide the ability to let parent components control layout of children components, the root element of child components also receives the data-attribute.

> _**Important note**: Styles are inherited and cascade as applied to the computed DOM and should not be confused with applying only to the Vue template file._

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
/* These cascade only to to children elements of this component */
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

## Deep Selectors

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

## Dynamically Generated Content

DOM content created with `v-html` are not affected by scoped styles, but you can still style them using deep selectors.

## True encapsulation via Shadow DOM

True encapsulation where styles do not leak in or out require custom web component elements which use Shadow DOM (not yet fully supported by browsers). Here is an example of how to add custom elements to your Vue application: https://github.com/karol-f/vue-custom-element.
## Keep in Mind

- **Understand CSS Cascading inheritance and specificity.**
    Styles are inherited and cascade as applied to the computed DOM and should not be confused with applying only to the Vue template file.
    - [Read about Cascade and Inheritance from Mozilla](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Cascade_and_inheritance)
    - [Read about CSS Inheritance from Sitepoint](https://www.sitepoint.com/css-inheritance-introduction)

- **Scoped styles do not eliminate the need for classes.**
    Due to the way browsers render various CSS selectors, `p { color: red }` will be many times slower when scoped (i.e. when combined with an attribute selector). If you use classes or ids instead, such as in `.example { color: red }`, then you virtually eliminate that performance hit. [Here's a playground](http://stevesouders.com/efws/css-selectors/csscreate.php) where you can test the differences yourself.

- **Be careful with descendant selectors in recursive components!**
    For a CSS rule with the selector `.a .b`, if the element that matches `.a` contains a recursive child component, then all `.b` in that child component will be matched by the rule.

- **You can prevent style conflicts:**
    To prevent conficting rules of parent and child components when CSS selector specificity or inheritance causes conflict:
    - Check the compiled style rules in the browser dev tools to determine the conflicting cascade **inheritance** and **specificity** on the computed DOM.
    - Avoid using selectors that resolve to the same element.
    - Use a wrapper node in your component:
        ``` html
        <!-- Vue component template -->
        <template>
          <div> <!-- << let this div be used by parent CSS for layout -->
            <div class="example">hi</div>
          </div>
        </template>
        ```
