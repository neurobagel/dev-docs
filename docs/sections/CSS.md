---
title: CSS
tags: [Front-end, Layout]

---

Given the visual nature of CSS and styling related tricks and strategies, we've decided to include resources as opposed to notes for this section so the following are the resources that proved helpful in making the query tool UI responsive

- [Learn CSS Grid - A 13 minute Deep Dive (YT video)](https://www.youtube.com/watch?v=EiNiSFIPIQE)
- [Learn Flexbox CSS in 8 minutes (YT video)](https://www.youtube.com/watch?v=phWxA89Dy94)
- [ A practical guide to responsive web design (YT video)](https://www.youtube.com/watch?v=x4u1yp3Msao)


## User-Agent Stylesheets

User-Agent stylesheet refers to the default CSS file that every browser is shipped with.
It contains the base rules that give HTML elements their initial appearance (margins, font sizes, colors, form-control styling, etc.).  
Since each browser (and even each version) can define these defaults differently, the same HTML may look slightly inconsistent across environments. Developers and libraries override User-Agent stylesheets for corss-browser consistency.

### Tailwind approach for addressing User-Agent stylesheets

Tailwind uses preflight which uses [modern-normalize](https://github.com/sindresorhus/modern-normalize) as a base, then applies it's own opinionated reset.

### MUI approach for addressing User-Agent stylesheets

Built on [nomralize.css](https://github.com/necolas/normalize.css) plus MUI specific-globals which is applied using their `CssBaseline` component.

### Tailwind and MUI interoperability

When using Tailwind with MUI the [MUI v6 docs](https://v6.mui.com/material-ui/integrations/interoperability/#tailwind-css) instruct users to turn off the Tailwind reset i.e., set `preflight` to `false` and instead use their `CssBaseline` to reset the User-Agent stylesheets.

Sources:
[MDN docs on CSS and user-agent stylesheets](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Cascade#user-agent_stylesheets)
[Tailwind preflight](https://tailwindcss.com/docs/preflight)
[MUI CssBaseline](https://mui.com/material-ui/react-css-baseline/)