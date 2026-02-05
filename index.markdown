---
layout: default
title: "Home"
---

I’m Pedro, a platform engineer obsessed with making things automated, scalable, and not break at 3 a.m. when no one wants to be on-call.

> Want to work together? Behold, [my résumé]({{ '/assets/files/cv/english_cv.pdf' | relative_url }}).

I work with **AWS** and **GCP**, using **Kubernetes**, **Terraform**, **Pulumi**, and friends (**ArgoCD**, **Flux**, **Helm**) to do real GitOps, not just slideware. For production visibility, I lean on **Prometheus**, **Grafana**, **Loki**, and **OpenTelemetry**. I’m finishing my Computer Science degree at UOC, and I want to bring that background together with my practical DevOps/SRE work to keep building reliable, large-scale systems.

I like to chat about basketball, the NFL, the NHL, and F1, especially when there are good rivalries or playoff drama going on. I’m also happy to get lost talking about guitars, pedals, tones, and why I still don’t play as much as I’d like. Throw in board games, strategy-heavy videogames, and I can easily turn “just one game” into a whole evening of nerding out.

---


This is where I write about real-world DevOps, SRE, incidents, CI/CD, and how to build modern infrastructure that not only works technically, but also fits the people and the team culture.     

<ul>
  {% for post in site.posts %}
    <li>
      {{ post.date | date_to_string }} – <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

