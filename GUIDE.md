# Site Management Guide

## 1. Adding a New Employee (Author)

To add a new person to the site:

1.  Create a new folder in `content/team/` with their name (e.g., `content/team/joca/`).
2.  Inside that folder, create an `_index.md` file.
3.  Add the following frontmatter and content:

```yaml
---
title: "Joca"
role: "Lead Researcher"
avatar: "/images/team/joca.jpg" # Make sure to upload the image here
socials:
  github: "https://github.com/joca"
  twitter: "https://twitter.com/joca"
---

This is the biography text. It will appear on the profile page.
```

4.  **Important:** The folder name (`joca`) is the ID you will use in blog posts to link them to this author.

## 2. Writing a Blog Post

1.  Create a new markdown file in `content/posts/` (e.g., `content/posts/my-new-experiment.md`).
2.  Use the following frontmatter:

```yaml
---
title: "My New Experiment"
date: 2026-01-01T12:00:00-00:00
draft: false
team: ["joca", "grimm"] # Use the folder names from step 1
---

Your post content goes here using standard Markdown.
```

## 3. Adding a Project

Projects work similarly to blog posts but live in `content/projects/`.

1.  Create a new markdown file in `content/projects/` (e.g., `content/projects/my-project.md`).
2.  Use the following frontmatter:

```yaml
---
title: "Project Name"
date: 2026-02-01T12:00:00-05:00
summary: "A brief description displayed on the card."
external_url: "https://project-url.com" # Where the card links to
team: ["joca"] # Attribution
team_project: false # Set to true to display on the main page
---
```

*   **team_project:** If `true`, the project appears in the "Selected Works" section of the homepage. If `false` (default), it only appears on the author's profile page.

## 4. Managing Images

*   **General Images:** Store them in `static/images/`.
*   **Author Avatars:** Store them in `static/images/team/`.
*   **Post Images:** You can store them in `static/images/posts/` or use page bundles.

To use an image in a post: `![Alt Text](/images/posts/image.png)`

## 5. Layouts

*   **Homepage:** `layouts/index.html`
*   **Single Post:** `layouts/_default/single.html`
*   **Employees List:** `layouts/team/terms.html`
*   **Employee Profile:** `layouts/team/list.html`
