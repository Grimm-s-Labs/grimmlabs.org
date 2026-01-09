---
title: "Building the New Grimm Labs Website"
date: 2026-01-08T10:00:00Z
draft: false
team: ["jocadbz"]
---

We finally managed to build the first version of the Grimm Labs website! This was a joint effort of all the team and I would like to thank everyone that helped me, specially with CSS advice (I'm not very skilled with it...)

Anyways, I would like to take a minute and explain to you how we built this

## The idea

At first, we had three basic requirements for the website:
* A site that would allow us to easily deploy written posts
* A site that could let us easily update information and add/remove content, authors, elements, etc
* A site that could be easily hosted.

Naturally, the first idea was to settle for a CMS (Content Management System) such as wordpress, Ghost, etc. But that was quickly put off the table due to the fact that hosting them would cost us money.
At this point we decided that the best course of action would be writing a site from scratch, but then a hiccup comes - It would require us to write all of our posts in HTML, and that is absolutely not an option (Ever wrote a big text in Raw HTML? I hope not.)

So, of course, we decided on using a static site generator (In our case, Hugo.). It allows us to write content in markdown and submit everything through PRs, which is acceptable and very good.

## Writing the website
Writing the actual site in Hugo is mostly a no-brainer. You just define your templates for pages using HTML and Go's templates and hugo takes care of the rest.

For example, our employee profile page's layout is mostly composed of code like this:

```html
<div class="bio" style="margin-bottom: 2rem; max-width: 600px; color: #ccc;">
    {{ .Content }}
</div>

<div class="socials">
    {{ range $name, $url := .Params.socials }}
    <a href="{{ $url }}" target="_blank" class="cta-button" style="margin-right: 1rem; font-size: 0.8rem; padding: 0.5rem 1rem; border: 1px solid var(--border-color); text-decoration: none; color: var(--text-main);">{{ $name }}</a>
    {{ end }}
</div>
```

Hugo basically interprets those layouts and dynamically generates content based on the markdown files we have for each team member. So, for example, my profile:
```md
---
title: "Jocadbz"
role: "Founder & Lead Researcher"
avatar: "/images/team/jocadbz.jpg"
socials:
  website: "https://jocadbz.xyz"
  X: "https://x.com/jocadbz"
---

Crazy ass idiot, also the founder of Grimm Labs.
```
Gets generated as:
```html
    <div class="bio" style="margin-bottom: 2rem; max-width: 600px; color: #ccc;">
        <p>Crazy ass idiot, also the founder of Grimm Labs.</p>

    </div>

    <div class="socials">
        
        <a href="https://jocadbz.xyz" target="_blank" class="cta-button" style="margin-right: 1rem; font-size: 0.8rem; padding: 0.5rem 1rem; border: 1px solid var(--border-color); text-decoration: none; color: var(--text-main);">website</a>
        
        <a href="https://x.com/jocadbz" target="_blank" class="cta-button" style="margin-right: 1rem; font-size: 0.8rem; padding: 0.5rem 1rem; border: 1px solid var(--border-color); text-decoration: none; color: var(--text-main);">x</a>
        
    </div>
</div>
</section>

<section id="author-posts">
<div class="section-header">
    <span class="mono-label">BLOG POSTS BY JOCADBZ</span>
</div>


<div class="grid">
    
    <div class="card">
        <div class="card-header">
            <h3><a href="/posts/first-transmission/" style="text-decoration: none; color: inherit;">Building the New Grimm Labs Website</a></h3>
        </div>
        <div class="meta" style="margin-bottom: 1rem; font-size: 0.8rem; color: var(--text-muted); font-family: var(--font-mono);">
            Jan 8, 2026
        </div>
        <p><p>We finally managed to build the first version of the Grimm Labs website! This was a joined effort of …</p></p>
    </div>
    
</div>
```
Pretty neat, right?

## Authoring posts

Next issue we had to tackle is the authorship of posts. How to actually assign a post to an specific team member? Initially, we figured that we would have to write things manually, but luckily, Hugo's got a feature for that.

**Taxonomy** of contents are basically tags you can assign to a specific page. Most people use them for separating articles into specific contents, but we currently use them to assign posts to team members.
First, you define the specific taxonomy in Hugo's config file:
```toml
[taxonomies]
  team = "team"
  tag = "tags"
  category = "categories"
```
After that, you assign it to a specific member:
```yaml
---
title: "Building the New Grimm Labs Website"
date: 2026-01-08T10:00:00Z
draft: false
team: ["jocadbz"]
---
```
And then you can filter it for each author in the team page:
```html
{{ if .Pages }}
<div class="grid">
    {{ range .Pages }}
    <div class="card">
        <div class="card-header">
            <h3><a href="{{ .RelPermalink }}" style="text-decoration: none; color: inherit;">{{ .Title }}</a></h3>
        </div>
        <div class="meta" style="margin-bottom: 1rem; font-size: 0.8rem; color: var(--text-muted); font-family: var(--font-mono);">
            {{ .Date.Format "Jan 2, 2006" }}
        </div>
        <p>{{ .Summary | truncate 100 }}</p>
    </div>
    {{ end }}
</div>
{{ else }}
<p style="color: var(--text-muted); font-family: var(--font-mono); margin-top: 1rem;">No blogposts yet.</p>
{{ end }}
```
Pretty cool!

## Conclusion

Now members can easily submit their blogposts and without having to code anything. Our github repo automatically rebuilds our hugo website whenever updated, so deploying is also automatic.
Anyways, thank you for reading, and expect more blogposts here, either by me, or other members of the lab. If you haven't already, join our [discord server](https://discord.grimmlabs.org)! We talk about computers and various other subjects.

Thank you for reading.
