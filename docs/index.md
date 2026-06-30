---
layout: default
title: Home
---

# Hey There 👋

<img src="/mario-funko.jpeg" alt="Mario Arce" width="200" style="border-radius: 10px; margin: 20px 0;" />

I'm **Mario**, and I've been building stuff on the cloud for 20+ years. This is where I share the real code, the actual patterns, and the things that worked (and didn't) in production.

You know how LinkedIn is all polished takes and Medium is long-form pontification? This is different. Think of it as the engineering channel on Slack—technical, unfiltered, peer-to-peer.

<p><br></p>

## What's Here

### 🏗️ Architectural Patterns
Real system designs from actual projects. Not textbook stuff—the messy reality of microservices, event-driven systems, and what happens when theory meets production at 3am.

### 💻 Production Code
The kind of code you actually ship. Custom implementations, solutions to gnarly problems, and the helper classes that save your sanity. De-branded and ready to steal.

### 🔬 PoCs & Experiments
When you need to prove something works before betting the sprint on it. Interactive demos, quick prototypes, and "let me show you what I mean" examples.

### 📚 PowerCSharp
My .NET open-source ecosystem. Built it because I got tired of writing the same boilerplate in every project. You probably have too.

<p><br></p>
---
<p><br></p>

## Latest Posts

<ul>
  {% for post in site.posts limit:5 %}
    <li>
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
      <h3><a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a></h3>
      <p>{{ post.excerpt }}</p>
    </li>
  {% endfor %}
</ul>

<p><br></p>
---
<p><br></p>

## Where Else to Find Me

I'm scattered across the usual spots:

**[LinkedIn](https://www.linkedin.com/in/marioalbertoarce/)** — The polished version. Career stuff, architectural hot takes, leadership thoughts. You know, the content your CTO follows.

**[Medium](https://medium.com/@marioarce)** — When I need more than 280 characters. System design deep-dives, modernization war stories, lessons from 20+ years of "oh crap" moments.

**[Hashnode](https://hashnode.com/@marioarce)** — Step-by-step technical guides. The kind you bookmark at 2am when production is on fire and you need answers.

**[DEV Community](https://dev.to/marioalbertoarce)** — Quick hits, community vibes, the occasional "TIL" post. More casual than Medium, less formal than LinkedIn.

**[GitHub](https://github.com/marioarce)** — Where the actual code lives. PowerCSharp and other OSS projects I maintain. PRs welcome, issues even more so.
