---
layout: default
title: Home
---

# Technical Living Lab

<img src="/mario-funko.jpeg" alt="Mario Arce" width="200" style="border-radius: 10px; margin: 20px 0;" />

I'm **Mario Alberto Arce**, an AWS Cloud Architect with 20+ years in software engineering. This is my technical execution layer—a living lab for architectural patterns, production-grade code, and deep technical expertise.

## What You'll Find Here

### 🏗️ Architectural Blueprints
System design patterns, microservices topologies, and event-driven architectures from real-world implementations.

### 💻 Code Showcases
Production-grade code samples, custom implementations, and practical solutions to complex engineering problems.

### 🔬 Living Labs
Interactive proofs-of-concept, executable specifications, and hands-on technical demonstrations.

### 📚 PowerCSharp Ecosystem
Documentation and resources for my .NET open-source projects focused on developer productivity and reusable patterns.

---

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

---

## About This Platform

This site serves as the **technical source of truth** in my content ecosystem:

- **LinkedIn / Medium** → High-level insights, architectural opinions
- **Hashnode** → Detailed technical breakdowns, system design
- **GitHub Pages** → Live PoCs, executable code, architectural blueprints (you are here)

**Target Audience:** Staff Engineers, Engineering Directors, CTOs, Senior Developers

**Tone:** Authoritative, deeply technical, pragmatic, trade-off focused

---

## Connect

- [LinkedIn](https://www.linkedin.com/in/marioalbertoarce/)
- [Medium](https://medium.com/@marioarce)
- [DEV Community](https://dev.to/marioalbertoarce)
- [GitHub](https://github.com/marioarce)
