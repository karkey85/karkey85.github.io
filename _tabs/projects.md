---
title: Projects
icon: fas fa-code
order: 4
---

<style>
.projects-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    margin-top: 30px;
}

.project-card {
    padding: 24px;
    border-radius: 16px;
    background: #1e1e1e;
    border: 1px solid #333;

    transition: transform 0.2s ease,
                border-color 0.2s ease,
                box-shadow 0.2s ease;
}

.project-card:hover {
    transform: translateY(-5px);

    border-color: #00ffff;

    box-shadow: 0 0 20px rgba(0,255,255,0.2);
}

.project-card a {
    text-decoration: none;
    color: inherit;
    display: block;
}

.project-title {
    font-size: 1.4rem;
    font-weight: bold;
    margin-bottom: 10px;
}

.project-desc {
    color: #b0b0b0;
    font-size: 0.95rem;
    line-height: 1.5;
}
</style>

# Projects

<div class="projects-grid">

<div class="project-card">
<a href="/projects/cursor-letter.html">

<div class="project-title">
✨ Cursor Effect
</div>

<div class="project-desc">
A minimal cursor-following alphabet effect using JavaScript.
</div>

</a>
</div>

<div class="project-card">
<a href="/projects/matrix-rain.html">

<div class="project-title">
🌧️ Matrix Rain
</div>

<div class="project-desc">
Matrix-style falling green character animation using Canvas.
</div>

</a>
</div>

<div class="project-card">
<a href="/projects/js-game.html">

<div class="project-title">
🎮 JS Game
</div>

<div class="project-desc">
A small browser game built with JavaScript and HTML5 Canvas.
</div>

</a>
</div>

</div>
