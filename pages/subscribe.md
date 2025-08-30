---
layout: page
title: Subscribe
permalink: /subscribe/
---

{% assign feed_url = site.url | append: site.baseurl | append: '/feed.xml' %}

## Subscribe

### RSS / Atom (recommended)

- Feed URL: <code>{{ feed_url }}</code>

<div class="subscribe-actions">
  <a class="btn primary" href="{{ '/feed.xml' | relative_url }}" type="application/atom+xml">Open RSS</a>
</div>

### Email (optional)

I don’t send a newsletter yet. If you want email alerts, use an RSS-to-email service (e.g., Follow.it, Feedrabbit, Blogtrottr) and paste the feed URL above.

<!-- Later: embed Buttondown/MailerLite form here -->
