---
layout: post
title:      "A Quick Refresher on CSS Specificity"
date:       2018-08-02 11:17:00 -0400
permalink:  a_quick_refresher_on_css_specificity
---

As I'm going through the interview process for a web development position, I've noticed that CSS specificity questions come up frequently for front-end positions. While I've used plenty of CSS in my personal projects, these projects tended to be smaller and contained, while likely any project at a new position is going to have much more complicated styling requirements. While working on my personal projects I wasn't required to take advantage of CSS specificity rules as often as I likely will in the real world, so when it came time for interviews, I thought I would review how they work. In truth, it's quite simple and the question of which CSS rule takes preferences (often found in hiring quizzes) can be easily answered once you understand specificity.

### The Specificity Hierarchy

Here is the ranking of CSS selectors from highest to lowest specificity. For simple rules, if an element matches more than one rule, you can just check which rule's selector has higher specificty to find out which will apply.

1. **Inline styles**:
  * e.g. `<p style="color: blue">`
2. **IDs**:
  * e.g. `<p id="paragraph-1">`
3. **Classes, attributes, and pseudo-classes**:
  * e.g. `<p class="intro">`
  * e.g. `<input type="text">`
  * `:hover`, `:focus`, etc.
4. **Elements and pseudo-elements**
  * `<p> <a> <div>`, etc.
  * `:before`, `:after`, etc.

### Calculating Specificity for a Rule
For CSS rules with multiple selectors, you can calculate the specificity of a rule with the following procedure. If an element matches multiple rules, the rule with the higher specificity will apply.

**To calculate specificity for a CSS rule:**
1. Start at 0
2. Add 1000 for in-line style attributes (this you can find in the HTML)
3. Add 100 for each ID selector in the rule
4. Add 10 for each attribute, class or pseudo-class selector in the rule
5. Add 1 for each element name or pseudo-element selector in the rule

The total is the final specificity value. The rule with the highest values will apply in the case that an element that matches multiple rules.

_**Important Note!** If 2 rules have the same specificty value, the rule that is defined later (lower in the file) will win out._

This also applies for different ways of including your CSS in your HTML file. Whether you write it directly in a `<style>` block, `<link>` to it, or `@import` it, whichever rules are loaded / declared last will overwrite previous rules, and in cases of tied specificity, will win out.

### Directly Targeted Element v. Inherited Styles
One slightly confusing part of all this is that in the case that an element inherits a style from a parent element (e.g. a `<p>` element inside of a `<div>`), if a CSS rule _directly targets_ the child element, that rule will take precedence over a rule that the child inherits from the parent, even if the parent rule has a higher specificity. This is best illustrated with an example.

_CSS:_
```
#parent {
  color: red;
}

p {
  color: blue;
}
```

_HTML:_
```
<div id="parent">
  <p>Hello!</p>
</div>
```

In this case, the text inside the `<p>` tags will actually appear as blue. This is because the `<p>` element was directly targeted, whereas the red styling defined for the parent's text color was an inherited value.

#### Check for `!important`
One thing to be aware of is that if a CSS rule has an  `!important` declaration, it will apply to any matched element, _regardless of whether there are other more specific rules that target that element._

For this reason, using `!important` in your styles is best avoided as it can make it very hard to understand why certain styles are being applied instead of others. One use case in which I have found `!important` to be useful is when needing to override certain CSS styles bundled into 3rd party libraries (e.g. Bootstrap).

### Summing Up
This article covered the basics of CSS specificity. There are some additional wrinkles and exceptions that come up that I haven't covered here, but this should get you through most interview questions. See the references below to learn more.

#### References:
- [MDN on CSS Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
- [CSS Precedence](https://css-tricks.com/precedence-css-order-css-matters/)
