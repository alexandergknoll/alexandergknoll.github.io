---
title: Why I use TailwindCSS despite the controversy
date: 2025-09-22 13:34:00
categories: [Development, Frontend]
tags: [development, frontend, html, css, tailwindcss]
---

## Introduction

As a heavy user of [TailwindCSS](https://tailwindcss.com/) since its first major release over 5 years ago, I'm often asked what makes it so great.  Typically, the question comes from more seasoned developers who can't understand why someone would want to go back to a practice that looks an awful lot like glorified [inline CSS styling](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/style).  It's hard for me to think of another single tool that has changed my approach and workflow so much, and at the same time I often struggle to explain to skeptics why I believe its adoption can be such an asset.

This post aims to explain what makes it so different from its predecessors, what makes it controversial and why, despite that controversy, I believe it's one of the best and most important tools created for frontend developers in the last decade.

## What makes TailwindCSS different?

A common question I receive about Tailwind is what makes it different from predecessors like [Bootstrap](https://getbootstrap.com/) and [Foundation](https://get.foundation/).  The first and most important thing to know is that Tailwind is not a component library; it does not contain any pre-built styles for commonly used UI elements such as buttons, cards, dialogs, or badges.  Instead, Tailwind leaves the creation of these components to you by joining together a list of completely unopinionated utility classes.  As such, it is really not a successor to either of these libraries at all which were, at their core, component libraries.

### Utility-first approach

Tailwind is fundamentally an extensible collection of atomic, single-purpose utility classes that can be used to style things without ever needing to leave your HTML to write any CSS.  This makes styling extremely predictable and eliminates the mental burden of context switching between HTML and CSS files.

### Design system architecture

In addition to implementing a well-thought-out base configuration for things like colors, spacing, and typography, the implementation of Tailwind really encourages you to think about your design system more holistically.  Working with Tailwind's configuration to include your own design tokens for these parameters allows you to scaffold all of the utility classes in a natural way that promotes consistency throughout your project.

### Purging of unused CSS

By default, Tailwind will parse the files in your project in any directories that you specify for Tailwind's classes and automatically purge any unused classes from your production stylesheet.  This ensures that you always have the smallest possible stylesheet.

### Extensibility and developer experience

Tailwind's configuration system makes it incredibly easy to extend the framework with your own custom utilities, variants, and design tokens. The developer experience is enhanced through excellent [IntelliSense](https://code.visualstudio.com/docs/editing/intellisense) support, comprehensive documentation, and a vibrant ecosystem of plugins and tools. The framework also provides powerful features like arbitrary value support (using square brackets like `w-[123px]`) and just-in-time compilation that generates styles on-demand as you author your templates.

## What's the controversy over?

You might be thinking that this all sounds great so far; what could be so controversial?  But it's important to recognize that adopting Tailwind does come with some drawbacks.  Writing your frontend with Tailwind represents a paradigm shift that challenges a lot of pretty deeply-held views of many developers.

### Verbose, non-semantic class naming

One of the most common criticisms of Tailwind is that it produces HTML that's difficult to read due to long strings of utility classes. Instead of semantic class names like `.navbar` or `.section-heading`, you'll see classes like `flex items-center justify-between px-4 py-2 bg-blue-500 text-white rounded-lg`. To many developers, this feels like a regression to the bad old days of inline styling, where presentation logic was mixed directly with markup.

### Separation of concerns

Traditional web development has long advocated for a clear separation of concerns: HTML for structure and hierarchy, CSS for presentation and styling, and JavaScript for behavior and functionality. Tailwind deliberately blurs the line between HTML and CSS, putting styling decisions directly in the markup. This challenges a fundamental principle that many developers have built their careers around, making it feel like we're abandoning best practices that took years to establish.

### Entrenched architecture patterns

Many developers have invested significant time learning CSS methodologies like [BEM (Block Element Modifier)](https://getbem.com/), [OOCSS (Object-Oriented CSS)](https://medium.com/@cssjhnnamae/an-introduction-to-object-oriented-css-oocss-0b9cd97ee038), and [SMACSS (Scalable and Modular Architecture for CSS)](https://smacss.com/). These systems provide structured approaches to organizing and naming CSS classes. Tailwind's utility-first approach sidesteps these methodologies entirely, which can feel like throwing away years of accumulated knowledge and best practices.

### Additional HTML maintenance challenges

Because HTML becomes the single source of truth for styling with Tailwind, it becomes very important to implement good component and partial HTML abstraction to avoid repetition of long strings of utility classes across your codebase. Without proper componentization, you can end up with unmaintainable HTML files filled with repeated class combinations that become a nightmare to update consistently.

## Code maintenance and other considerations

Although these are all valid concerns, I believe the benefits of Tailwind far outweigh the drawbacks, especially when it comes to long-term code maintenance and developer productivity.

### Clear intent & consistent patterns

With Tailwind, you can immediately see what styles are applied just by looking at the HTML without hunting through CSS files. There's no mystery about where a particular style is coming from or what CSS rule might be overriding another. The styling is explicit and co-located with the markup, making it much easier to understand the visual structure of a component at a glance.

### Refactoring with confidence

In the traditional approach to CSS, if you have a class like `.card` and you decide to change some properties on it, you might break a component somewhere that you didn't even know was using that style. The larger and more complex a project becomes, the more risk and fear you experience every time you touch something in a stylesheet. Any change to your CSS carries a risk of causing unintended and unpredictable consequences throughout your codebase.

With the Tailwind approach, you know with 100% confidence that by modifying utility classes on your HTML element your change won't carry unintended consequences beyond that explicit piece of HTML.

### No dead CSS

With Tailwind's utility-first approach and built-in purging, you never accumulate dead CSS. Every class that appears in your final stylesheet is actually being used somewhere in your codebase.  Traditional CSS approaches often lead to bloated stylesheets filled with unused rules due to the fear of breaking something by removing those styles.

### No more ever-growing stylesheets

Because you effectively stop writing CSS in stylesheets when you style with Tailwind, and because unused utilities are automatically purged from your production build, your CSS bundle size remains predictably small and doesn't grow over time. Traditional CSS architectures often suffer from continuously growing stylesheet files that become harder to maintain as projects mature.

### Speed & agility of working with established codebases

If you're familiar with Tailwind at even a fairly basic level, you can easily and confidently navigate and implement changes to other projects that use Tailwind without the burden of having to familiarize yourself with a project's custom CSS architecture. The utility classes are standardized across all Tailwind projects, so your knowledge transfers seamlessly from one codebase to another. This dramatically reduces the ramp-up time for new developers joining a project and makes context switching between different Tailwind-based projects much more efficient.

## Conclusion

While I acknowledge that TailwindCSS isn't perfect and that the criticisms leveled against it have merit, I believe it represents a significant step forward in how we approach CSS architecture. The utility-first methodology addresses real pain points that have plagued CSS development for years: unpredictable cascade behavior, ever-growing stylesheets, fear-driven refactoring, and the cognitive overhead of maintaining complex CSS architectures.

The apparent regression to "inline styling" is actually a carefully designed system that provides the benefits of inline styles (predictability, locality, explicit intent) while avoiding the drawbacks (repetition, maintainability issues, design inconsistency) through its systematic approach to design tokens and utility generation.

Yes, it requires a shift in thinking and yes, it challenges conventional wisdom about separation of concerns. But in my experience, once you embrace the utility-first mindset and establish good patterns for component abstraction, you'll find yourself building UIs faster, with more confidence, and with significantly less technical debt than traditional CSS approaches.

The controversy around Tailwind often stems from its challenge to established practices rather than fundamental flaws in the approach. For teams willing to embrace this paradigm shift, I believe TailwindCSS offers one of the most productive and maintainable ways to handle styling in modern web development.

**Questions or comments?** Sign in with GitHub to comment below!
