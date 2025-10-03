# CSS-APPLY

CSS-APPLY is a way to apply entire sets of variables to a selector in a single step.

The main idea is to completely transfer the design logic to CSS and abstract it from traditional HTML declarations.

Currently, the `.class` selector is primarily used to allow a custom component to inherit a complete style.

Otherwise, all the main styles must be carefully copied into the element's selector, generating duplicate data and potential synchronization issues.

CSS-APPLY makes this behavior easier. It's like applying a `.class` directly to a selector.

```css
--main-them {
background-color: blue;
}
--theme-box {
box-shadow: black;
}

custom-element {
apply: set(--main-theme);
apply: set(--theme-box);
}

button-element {
apply: set(--main-theme);
apply: set(--theme-box)
}
```

The great thing is that CSS-APPLY is completely natural. It doesn't require any modifications to the CSS engine. A simple parsing above solves this.

As you can see, it also follows the expected flow of CSS variables. Placing an apply is exactly the same as saying "put these variables" here.

Currently, CSS-APPLY is working, has been tested in demanding scenarios, and is being applied in various projects.

I hope CSS-APPLY can be considered for standard implementation by the W3C.
