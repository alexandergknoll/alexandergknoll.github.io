---
title: Arbitrary values & variants in TailwindCSS
date: 2025-10-07 17:09:00
categories: [Development, Frontend]
tags: [development, frontend, html, css, tailwindcss]
---

[TailwindCSS](https://tailwindcss.com/) has several lesser-known advanced features that can be useful for handling edge cases that many believe would require you to add custom CSS to a stylesheet. In fact, it's possible to handle these without having to ever write a single line of CSS in your stylesheet.

## 1. Arbitrary values

Tailwind supports [arbitrary values](https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values) in utility classes which allows you to create custom one-off classes without writing any CSS in a stylesheet.

### An example

Consider a card component built with Tailwind utility classes:

```html
<div class="bg-white text-gray-800 rounded-lg shadow-lg p-6">
  <h2 class="text-xl font-bold mb-3">Ut enim ad minim veniam</h2>
  <p class="text-base leading-relaxed">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
  </p>
</div>
```

This card works great throughout your project. But on one page, you need a background color of `#dbdbdb` and a width of `410px` (neither of which are values that are part of Tailwind's default configuration). These one-off styles are needed for this page only and are very unlikely to be needed anywhere else in the project.

Without using Tailwind's arbitrary values feature, there are a few possible solutions, all of which come with drawbacks.

#### Worst solution: write a custom CSS class

The most obvious solution is to just write a custom class and add it to your stylesheet:

```css
@layer components {
  .special-card {
    background-color: #dbdbdb;
    width: 410px;
  }
}
```

> The `@components` layer sits between `@theme` and @utility in Tailwind's cascade layer system, meaning component styles have higher specificity than theme/base styles but lower than utilities
{: .prompt-info }

```html
<div class="special-card rounded-lg shadow-lg p-6">
  <h2 class="text-xl font-bold mb-3">Ut enim ad minim veniam</h2>
  <p class="text-base leading-relaxed">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
  </p>
</div>
```

This works in a pinch, but *feels wrong* when it comes to Tailwind's core principles. It suffers from some of the issues I discussed in my [first post about TailwindCSS](/posts/why-i-use-tailwindcss-despite-the-controversy/):

1. Tightly coupled HTML and CSS that both need to be maintained in your codebase
2. The `.special-card` class is not automatically purged from your stylesheet if you remove it from your HTML in the future, potentially leading to the accumulation of [dead CSS](/posts/why-i-use-tailwindcss-despite-the-controversy/#no-dead-css)
3. Maintenance confusion and additional mental burden of [context switching](/posts/why-i-use-tailwindcss-despite-the-controversy/#utility-first-approach) between HTML and CSS: if I need to change the color or width in the future, do I need to go to the HTML and apply a new utility class, or update the hex value of the color in the CSS?

#### Better solution: extend Tailwind's configuration

By extending Tailwind using theme variables, we can scaffold its built-in utility classes for colors and spacing with some custom values. In this case, we can use the `--color-*` and `--spacing-*` namespaces for variables with the name `card` to create new CSS classes of `.bg-card` and `.w-card`:

```css
@theme {
  --color-card: #dbdbdb;
  --spacing-card: 410px;
}
```

> Custom variables that extend TailwindCSS via a prescribed namespace like `--color-*` or `--spacing-*` need to be declared in the @theme layer
{: .prompt-info }

```html
<div class="bg-card w-card text-gray-800 rounded-lg shadow-lg p-6">
  <h2 class="text-xl font-bold mb-3">Ut enim ad minim veniam</h2>
  <p class="text-base leading-relaxed">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
  </p>
</div>
```

This is a more "Tailwind" way of solving the problem and does at least allow for the purging of unused CSS by extending Tailwind's configuration. If removed from the HTML markup, the `.bg-card` CSS class is automatically purged from the stylesheet, for example.

One drawback to this solution is that we've extended these custom values to other utility classes where it might not make sense to have them. For example, with this configuration you would also have utility classes for padding and margins of 410px using `p-card` or `m-card`. One of the core principles of extending Tailwind's configuration in this way is to use it as a tool for creating a [design system](/posts/why-i-use-tailwindcss-despite-the-controversy/#design-system-architecture) for your project, which promotes consistency of tokens for colors, spacing, and other styles that create a cohesive look & feel. In that way, polluting that configuration with one-off tokens that don't have any meaning in the broader context of your project's design system isn't ideal.

#### Best solution: use arbitrary values with Tailwind's utility classes

```html
<div class="bg-[#dbdbdb] w-[410px] text-gray-800 rounded-lg shadow-lg p-6">
  <h2 class="text-xl font-bold mb-3">Ut enim ad minim veniam</h2>
  <p class="text-base leading-relaxed">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
  </p>
</div>
```

The best solution is to use Tailwind's arbitrary values feature which allows you to specify exact CSS values inline using square brackets. This works with virtually any Tailwind utility: `text-[14px]`, `top-[117px]`, `grid-cols-[200px_minmax(900px,_1fr)_100px]` (admittedly, it is **extremely ugly** in some cases). The generated CSS is still automatically purged when removed from your HTML.

This is the ideal solution for one-off styles that don't fit into your design system. You get all the benefits of Tailwind's utility-first approach without needing to extend your configuration or write custom CSS in a stylesheet.

## 2. Arbitrary variants

Tailwind supports using [arbitrary variants](https://tailwindcss.com/docs/hover-focus-and-other-states#using-arbitrary-variants) to apply styles based on custom selectors. This is particularly valuable when working with third-party libraries, content management systems, or any situation where you don't have direct control over the HTML markup being rendered.

### An example

When integrating third-party libraries, you often can't control the injected HTML. For example, a rich text editor might render something like this:

```html
<div class="wysiwyg-content">
  <!-- Arbitrary WYSIWYG content here that we don't control -->
  <h1>Lorem ipsum dolor sit amet</h1>
  <p>Consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.</p>
  <ul>
    <li>Ut enim ad minim veniam</li>
    <li>Quis nostrud exercitation ullamco</li>
    <li>Laboris nisi ut aliquip</li>
  </ul>
  <a href="#">Ex ea commodo consequat</a>
</div>
```

You have no control over the markup structure or the classes applied to these elements which are generated by the third-party library. But you need to style them to match your site's design. Without using Tailwind's arbitrary variants feature, there are a few possible solutions, all of which have drawbacks.

#### Worst solution: write custom CSS targeting the injected elements

The most obvious solution is to write CSS in your stylesheet using selectors that target the elements within the container:

```css
.wysiwyg-content h1 {
  font-size: 2.25rem;
  font-weight: 700;
  margin-bottom: 1rem;
}

.wysiwyg-content p {
  margin-bottom: 1rem;
  line-height: 1.75;
}

.wysiwyg-content ul {
  list-style-type: disc;
  margin-left: 1.5rem;
  margin-bottom: 1rem;
}

.wysiwyg-content li {
  margin-bottom: 0.5rem;
}

.wysiwyg-content a {
  color: #3b82f6;
  text-decoration: underline;
}
```

This works, but it completely abandons Tailwind's utility-first approach and brings back all the problems we were trying to avoid:

1. You're writing traditional CSS in a separate stylesheet, requiring context switching between files
2. These styles won't be automatically purged if you remove the container from your HTML
3. You lose the consistency and constraints of your design system by hardcoding numbers and hex values
4. The cascade and specificity can become complex and hard to reason about as your stylesheet grows

#### Better, incomplete solution: use Tailwind's child selector variants

Tailwind v4 introduced built-in child selector variants that allow you to target direct children. You can use the `*:` variant to target all direct children or `**:` to target all descendents.

```html
<div class="*:mb-4 *:leading-relaxed *:text-neutral-800">
  <!-- Arbitrary WYSIWYG content here that we don't control -->
  <h1>Lorem ipsum dolor sit amet</h1>
  <p>Consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.</p>
  <ul>
    <li>Ut enim ad minim veniam</li>
    <li>Quis nostrud exercitation ullamco</li>
    <li>Laboris nisi ut aliquip</li>
  </ul>
  <a href="#">Ex ea commodo consequat</a>
</div>
```

This is much better: we're back to using Tailwind utilities and staying within our design system. However, the built-in child selectors are very limited and we can't apply styles to specific types of elements. And you won't be able to target nested elements (like the `<li>` elements inside the `<ul>`), or use more complex selectors. So this only gets us part of the way.

#### Best solution: use arbitrary variants with Tailwind's utility classes

The best solution is to use Tailwind's arbitrary variants feature. Arbitrary variants allow you to write any CSS selector you need by wrapping it in square brackets and using the `&` symbol as a placeholder for the element you're styling:

```html
<div class="[&_h1]:text-4xl [&_h1]:font-bold [&_h1]:mb-4 [&_p]:mb-4 [&_p]:leading-relaxed [&_ul]:list-disc [&_ul]:ml-6 [&_ul]:mb-4 [&_li]:mb-2 [&_a]:text-blue-600 [&_a]:underline">
  <h1>Lorem ipsum dolor sit amet</h1>
  <p>Consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.</p>
  <ul>
    <li>Ut enim ad minim veniam</li>
    <li>Quis nostrud exercitation ullamco</li>
    <li>Laboris nisi ut aliquip</li>
  </ul>
  <a href="#">Ex ea commodo consequat</a>
</div>
```

The `&` symbol represents the element with the class, so `[&_h1]:text-4xl` generates CSS like `.className h1 { font-size: 2.25rem; }`. This gives you complete flexibility while maintaining all the benefits of Tailwind's utility-first approach.

You can use arbitrary variants for even more complex scenarios:

- `[&:nth-child(3)]:font-bold` - targets the third child
- `[&[data-active="true"]]:bg-blue-500` - targets elements with specific attributes
- `[&_.custom-class]:text-red-500` - targets elements with specific class names
- `[&:has(>img)]:p-0` - targets elements that contain an image
- `[&:not(:last-child)]:mb-4` - targets all but the last child

This approach keeps everything in your HTML, uses your design system tokens, automatically purges unused styles, and gives you the full power of CSS selectors when you need it. It's the perfect solution for styling third-party content, legacy markup, or any situation where you can't directly control the HTML structure.

## Conclusion

Arbitrary values and arbitrary variants are two powerful features that make TailwindCSS even more flexible than it might first appear. While Tailwind's design system constraints are valuable for maintaining consistency, these features give you an escape hatch when you need one-off styles or need to work with markup you can't control, all while staying within the utility-first paradigm.

**Questions or comments?** Sign in with GitHub to comment below!