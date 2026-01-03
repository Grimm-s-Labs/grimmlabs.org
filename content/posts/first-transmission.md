---
title: "Building the New Grimm Labs Website"
date: 2026-01-01T10:00:00Z
draft: false
team: ["jocadbz"]
---

We've officially launched the new Grimm Labs website. In the spirit of our lab's philosophy—simplicity and control—we decided against using heavy JavaScript frameworks for a content-heavy site. Instead, we chose **Hugo**.

## The Stack

*   **Core:** [Hugo](https://gohugo.io/) (Static Site Generator)
*   **Styling:** Raw CSS variables (No Tailwind, no Bootstrap).
*   **Typography:** *Inter* for UI, *JetBrains Mono* for data.
*   **Hosting:** Static deployment.

## Technical Highlights

### Custom Team Taxonomy
One of the key requirements was a robust system to credit multiple researchers on a single project. We implemented a custom Hugo **Taxonomy** called `team`.

This allows us to link posts to team members dynamically:

```yaml
---
title: "Project Alpha"
team: ["jocadbz", "contributor"]
---
```

When the site builds, Hugo automatically generates:
1.  A collective **Team Page** listing all active members.
2.  Individual **Profile Pages** for each member, aggregating every post they've contributed to.

### Minimalist Design
We stripped away the noise. The site uses a "dark mode by default" palette (`#030303`) with deep purple accents (`#7939a0`) to maintain readability while keeping the lab's visual identity. There is no client-side routing, no hydration errors, and no unnecessary bloat. Just HTML and CSS.

This infrastructure is now ready to document our future experiments.
