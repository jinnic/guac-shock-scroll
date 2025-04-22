---
title: 01. Section Detaction
toc: false
---


<!-- Scroll container -->
<section class="scroll-container">
  <div class="scroll-info"></div>
  <div class="scroll-section" data-step="1">STEP 1</div>
  <div class="scroll-section" data-step="2">STEP 2</div>
  <div class="scroll-section" data-step="3">STEP 3</div>
  <div class="scroll-section" data-step="4">STEP 4</div>
</section>

```js
// 6) OBSERVER: swap sections on scroll
const indicator = document.querySelector('.scroll-info');
const sections  = document.querySelectorAll('.scroll-section');

const observer = new IntersectionObserver((entries) => {
    // find the section with the largest intersection ratio
    const visible = entries
    .filter(e => e.isIntersecting)
    .sort((a, b) => b.intersectionRatio - a.intersectionRatio)[0];

    if (visible) {
      const step = visible.target.dataset.step;
      // update the text
      indicator.textContent = 'Section: ' + step;
      // switch background class
      indicator.className = `scroll-info scroll-info--step-${step}`;
    } else {
      indicator.textContent = 'Section: 0';
      indicator.className = 'scroll-info';
    }
}, {
    rootMargin: '-50% 0px -50% 0px',
    threshold: [0, 0.5, 1]
});

sections.forEach(sec => observer.observe(sec));
```


<style>
.scroll-container {
  position: relative;
  margin: 1rem auto;
  font-family: var(--sans-serif);
}

.scroll-info {
  position: sticky;
  aspect-ratio: 16 / 9;
  top: calc((100% - 9 / 16 * 100vw) / 2);
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 64px;
  transition: ease background-color 0.5s;
  background-color: var(--theme-background-alt);
}

.scroll-info--step-1 {
  background-color: #4269d0;
}

.scroll-info--step-2 {
  background-color: #efb118;
}

.scroll-info--step-3 {
  background-color: #ff725c;
}

.scroll-info--step-4 {
  background-color: #6cc5b0;
}

.scroll-section {
  position: relative;
  aspect-ratio: 16 / 9;
  margin: 1rem 0;
  display: flex;
  align-items: start;
  justify-content: center;
  border: solid 1px var(--theme-foreground-focus);
  background: color-mix(in srgb, var(--theme-foreground-focus) 5%, transparent);
  padding: 1rem;
  box-sizing: border-box;
}
</style>
