---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

<section class="hero" aria-labelledby="intro-title">
  <p class="eyebrow">Welcome, I'm Thotogelo</p>
  <h1 id="intro-title">Thinking out loud,<br><em>one note at a time.</em></h1>
  <p class="hero__summary">A small corner of the internet for ideas, experiments, and the things I'm learning along the way.</p>
  <div class="hero__actions">
    <a class="button button--primary" href="{{ '/about/' | relative_url }}">A little about me <span aria-hidden="true">→</span></a>
    <a class="button button--quiet" href="#writing">Explore the writing</a>
  </div>
</section>

<section class="link-grid" aria-label="Explore the site">
  <a class="link-card" href="{{ '/feel/' | relative_url }}">
    <span class="link-card__icon" aria-hidden="true">✦</span>
    <span><strong>Feel</strong><small>A visual pause</small></span>
    <span class="link-card__arrow" aria-hidden="true">↗</span>
  </a>
  <a class="link-card" href="{{ '/scripts/' | relative_url }}">
    <span class="link-card__icon" aria-hidden="true">&lt;/&gt;</span>
    <span><strong>Scripts</strong><small>Useful little tools</small></span>
    <span class="link-card__arrow" aria-hidden="true">↗</span>
  </a>
</section>

<div id="writing" class="section-heading">
  <p class="eyebrow">From the notebook</p>
  <h2>Recent writing</h2>
</div>
