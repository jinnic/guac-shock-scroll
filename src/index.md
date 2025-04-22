---
style: cfr-style.css
---

```js
import * as d3 from "npm:d3";
```

<!-- Scroll container -->
<section class="scroll-container">
  <div class="scroll-info">
    <!-- testing here -->
    <!-- Chart containers for responsive display -->
    <div id="chart-container" class="chart ">
      <div class="test">
          <p></p>
      </div>
      <h1>🥑</h1>
      <div id="chart-mobile" ></div>
      <div id="chart-tablet" ></div>
      <div id="chart-desktop" class="card"></div>
    </div>
  </div>
  <div class="scroll-section" data-step="1">
    <h1>section 1</h1>
    <!-- Text container – these are the narrative text sections -->
    <div class="text-section" data-group="groupA">
      <div class="text-box">
        Let’s say that before tariffs, the fictional company “U.S. Grocery” was importing avocados from Mexico for <span class="text-avocado">&nbsp;$1.00&nbsp;</span>, then selling them to U.S. consumers for <span class="text-consumer">&nbsp;$1.50&nbsp;</span>.
      </div>
    </div>
  </div>
  <div class="scroll-section" data-step="2">
    <h1>section 2</h1>
    <div class="text-section" data-group="groupB">
      <div class="text-box">
        If the U.S. government places a 25 percent tariff on Mexican goods, U.S. Grocery must pay a <span class="text-tariff">&nbsp;$0.25 tax&nbsp;</span> to the U.S. government for each avocado (25 percent of $1.00).<br><br>If U.S. Grocery wants to continue making <span class="text-profit">&nbsp;$0.50 in profit&nbsp;</span>, it must raise the price of avocados from $1.50 to <span class="text-consumer">&nbsp;$1.75&nbsp;</span>, “passing on” the $0.25 tariff to consumers. That is what usually happens, according to most economists.
      </div>
    </div>
  </div>
  <div class="scroll-section" data-step="3">
    <h1>section 3</h1>
    <div class="text-section" data-group="groupC">
      <div class="text-box">
        Alternatively, U.S. Grocery could decide to reduce its <span class="text-profit">&nbsp;profit&nbsp;</span> in order to keep its price at <span class="text-consumer">&nbsp;$1.50&nbsp;</span>. To restore its profits, U.S. Grocery would have to cut costs elsewhere, including by lowering wages, laying off workers, or investing less in its business.
      </div>
    </div>
  </div>
  <div class="scroll-section" data-step="4">
    <h1>section 4</h1>
    <div class="text-section" data-group="groupD">
      <div class="text-box">
        If U.S. Grocery could replace its Mexican supplier with a U.S. supplier, it would avoid the tariff. This is one goal of tariffs: to support U.S. companies struggling to compete with cheaper imports.<br><br>But  <span class="text-avocado">&nbsp;U.S. suppliers’ prices&nbsp;</span> are likely to be significantly higher—generally because of higher labor costs and production standards—and switching suppliers is difficult and expensive in practice. 
      </div>
    </div>
  </div>
</section>

```js
const data = FileAttachment("data/guac-price.csv").csv({ typed: true });
```

```js
data.forEach(d => {
  d.cost    = +d.cost;
  d.tariff  = +d.tariff;
  d.profit  = +d.profit;
  d.section = +d.section;
  d.total   = +d.total;
});

// Chart dimensions and keys
const width  = 100,
      height = 300;
const margin = { top: 20, right: 20, bottom: 20, left: 20 };
const keys   = ["cost", "tariff", "profit"];

// Color scale
const color = d3.scaleOrdinal()
  .domain(keys)
  .range(["#507a3a", "#a6bf43", "#d1601d"]);

// Create the SVG once
const svg = d3.create("svg")
  .attr("viewBox", [0, 0, width, height]);
document.getElementById("chart-desktop").appendChild(svg.node());

// X & Y scales 
const x = d3.scaleBand()
  .domain(["bar"])
  .range([margin.left, width - margin.right])
  .padding(0.2);

const y = d3.scaleLinear()
  .range([height - margin.bottom, margin.top]);

// Helper to compute the stacked segments by keys for a given section
function getSegments(sectionNum) {
  // D3.stack on the single-row array [ row ]
  const row = data.find(d => d.section === sectionNum);
  const series = d3.stack().keys(keys)([row]);

  // Flatten to {key, y0, y1, value} for each segment
  return series.map(layer => ({
    key:   layer.key,
    y0:    layer[0][0],
    y1:    layer[0][1],
    value: row[layer.key],
    total: row.total
  }));
}
// Initialize: draw one bar (all segments collapsed at y=0)
function initChart() {
  const segs = getSegments(1);
  const total = segs[0].total;

  console.log("segs", segs);
  const maxY = d3.max(segs, d => d.y1);
  y.domain([0, maxY]).nice();

  svg.selectAll("rect")
    .data(segs, d => d.key)
    .enter().append("rect")
      .attr("x",      x("bar"))
      .attr("width",  x.bandwidth())
      .attr("y",      y(0))
      .attr("height", 0)
      .attr("fill",   d => color(d.key))
    .transition().duration(800).ease(d3.easeCubicOut)
      .attr("y",      d => y(d.y1))
      .attr("height", d => y(d.y0) - y(d.y1));

  // draw value labels inside each segment
  svg.append("g").attr("class","values")
    .selectAll("text")
    .data(segs, d => d.key)
    .enter().append("text")
      .attr("x",      x("bar") + x.bandwidth()/2)
      .attr("y",      y(0))
      .attr("fill",   "white")
      .attr("font-size","10px")
      .attr("text-anchor","middle")
      .attr("opacity", 0)
    .text(d => `$${d.value.toFixed(2)}`)
    .transition().duration(800).delay(200).ease(d3.easeCubicOut)
      .attr("y",      d => {
        const mid = y(d.y0) - (y(d.y0) - y(d.y1))/2;
        return mid + 4;
      })
      .attr("opacity", 1);
  
  // total label on top
  svg.append("g").attr("class", "totals")
    .select("g.totals").selectAll("text")
    .data([segs[segs.length -1]], d => d.total)
    .join(
      enter => enter.append("text")
        .attr("x",        x("bar") + x.bandwidth()/2)
        .attr("y",        y(0))
        .attr("text-anchor","middle")
        .attr("font-weight","bold")
        .attr("opacity",  0)
        .text(d => `$${d.total.toFixed(2)}`)
        .call(ent => ent.transition().duration(800).delay(400)
          .attr("y",      d => y(d.y1) - 10)
          .attr("opacity",1)
        ),

      update => update.transition().duration(800).delay(400)
        .attr("y",    d => y(d.y1) - 10)
        .text(d => `$${d.total.toFixed(2)}`),

      exit => exit.remove()
    );
}

// 7) Update: rebind those same rects & animate their y/height
function updateChart(sectionNumber) {
  const sec  = +sectionNumber;
  const segs = getSegments(sec);
  const total = segs[0].total;

  const maxV = d3.max(segs, d => d.y1);
  y.domain([0, maxV]).nice();

  svg.selectAll("rect")
    .data(segs, d => d.key)
    .join(
      enter => enter.append("rect")
        .attr("x",      x("bar"))
        .attr("width",  x.bandwidth())
        .attr("y",      y(0))
        .attr("height", 0)
        .attr("fill",   d => color(d.key))
        .call(ent => ent.transition().duration(800).ease(d3.easeCubicOut)
          .attr("y",      d => y(d.y1))
          .attr("height", d => y(d.y0) - y(d.y1))
        ),

      update => update.transition().duration(800).ease(d3.easeCubicOut)
        .attr("y",      d => y(d.y1))
        .attr("height", d => y(d.y0) - y(d.y1)),

      exit => exit.transition().duration(300)
        .attr("y",      y(0))
        .attr("height", 0)
        .remove()
    );

  // update value labels
  svg.select("g.values").selectAll("text")
  .data(segs, d => d.key)
  .join(
    // ENTER: append at the bottom, zero opacity
    enter => enter.append("text")
      .attr("class",       "value")
      .attr("x",           x("bar") + x.bandwidth()/2)
      .attr("y",           y(0))
      .attr("fill",        "white")
      .attr("font-size",   "10px")
      .attr("text-anchor", "middle")
      .attr("opacity",     0)
      .text(d => `$${d.value.toFixed(2)}`)
      .call(g => g.transition()
        .duration(800)
        .delay(200)
        .ease(d3.easeCubicOut)
        .attr("y",      d => {
          const mid = y(d.y0) - (y(d.y0) - y(d.y1)) / 2;
          return mid + 4;
        })
        .attr("opacity", 1)
      ),

    // UPDATE: reposition and full opacity
    update => update
      .text(d => `$${d.value.toFixed(2)}`)   // update the text
      .transition()
        .duration(800)
        .delay(200)
        .ease(d3.easeCubicOut)
        .attr("y",      d => {
          const mid = y(d.y0) - (y(d.y0) - y(d.y1)) / 2;
          return mid + 4;
        })
        .attr("opacity", 1),                  // <— make sure we go back to 1

    // EXIT: fade out cleanly
    exit => exit.transition()
      .duration(300)
      .attr("opacity", 0)
      .remove()
  );
    
  svg.select("g.totals").selectAll("text.total")
  .data([segs[segs.length-1]], d => d.total)
  .join(
    // ENTER: start at y(0), opacity 0, then rise & fade in
    enter => enter.append("text")
      .attr("class",       "total")
      .attr("x",           x("bar") + x.bandwidth()/2)
      .attr("y",           y(0))
      .attr("text-anchor", "middle")
      .attr("font-weight", "bold")
      .attr("opacity",     0)
      .text(d => `$${d.total.toFixed(2)}`)
      .call(g => g.transition()
        .duration(800)
        .delay(400)
        .ease(d3.easeCubicOut)
        .attr("y",      d => y(d.y1) - 10 )
        .attr("opacity", 1)
      ),

    // UPDATE: move to new y, reset opacity to 1, update text
    update => update
      .text(d => `$${d.total.toFixed(2)}`)
      .transition()
        .duration(800)
        .delay(400)
        .ease(d3.easeCubicOut)
        .attr("y",      d => y(d.y1) - 10)
        .attr("opacity", 1),

    // EXIT: remove if ever needed
    exit => exit.remove()
  );
}


/*
// To Create multiple stack bar diffrent x position.
// INITIAL DRAW: build the three colored layers for section 1
function initChart() {
  const firstData = data.filter(d => d.section === 1);
  const series   = d3.stack().keys(keys)(firstData);
  const max0     = d3.max(series[series.length - 1], d => d[1]);
  y.domain([0, max0]).nice();

  // One <g> per key
  const layers = svg.selectAll("g.layer")
    .data(series, d => d.key)
    .enter().append("g")
      .attr("class", "layer")
      .attr("fill", d => color(d.key));

  // One <rect> per layer for section 1, but start collapsed at y(0)
  layers.selectAll("rect")
    .data(d => d, d => d.data.section)
    .enter().append("rect")
      .attr("x",       d => x(d.data.section))
      .attr("width",   x.bandwidth())
      .attr("y",       y(0))
      .attr("height",  0)
    .transition().duration(800).ease(d3.easeCubicOut)
      .attr("y",      d => y(d[1]))
      .attr("height", d => y(d[0]) - y(d[1]));
}

// UPDATE: stretch/shrink the same bars in place for any section
function updateChart(sectionNumber) {
  const sec = +sectionNumber;
  const currentData = data.filter(d => d.section === sec);
  if (currentData.length === 0) return;

  const series   = d3.stack().keys(keys)(currentData);
  const maxValue = d3.max(series[series.length - 1], d => d[1]);
  y.domain([0, maxValue]).nice();

  // Rebind each layer to its new stack values
  svg.selectAll("g.layer")
    .data(series, d => d.key)
    .selectAll("rect")
    .data(d => d, d => d.data.section)
    .join(
      // ENTER (shouldn't happen if you've init for all layers)
      enter => enter.append("rect")
        .attr("x",       d => x(d.data.section))
        .attr("width",   x.bandwidth())
        .attr("y",       y(0))
        .attr("height",  0)
        .call(enter =>
          enter.transition().duration(800).ease(d3.easeCubicOut)
            .attr("y",      d => y(d[1]))
            .attr("height", d => y(d[0]) - y(d[1]))
        ),

      // UPDATE: stretch/shrink in place
      update => update.transition().duration(800).ease(d3.easeCubicOut)
        .attr("x",       d => x(d.data.section))
        .attr("y",       d => y(d[1]))
        .attr("height",  d => y(d[0]) - y(d[1])),

      // EXIT: collapse and remove
      exit => exit.transition().duration(300)
        .attr("y",      y(0))
        .attr("height", 0)
        .remove()
    );
}
*/

// Kick things off
initChart();
updateChart(1);

// Set up IntersectionObserver to call updateChart on scroll
const currentSectionText = document.querySelector(".test");
const chart = document.querySelector(".chart");
const targets = document.querySelectorAll(".scroll-section");

const observer = new IntersectionObserver((entries) => {
  let best = null;
  for (const e of entries) {
    if (e.isIntersecting && (!best || e.intersectionRatio > best.intersectionRatio)) {
      best = e;
    }
  }
  if (best) {
    const step = best.target.dataset.step;
    currentSectionText.textContent = step;
    chart.className = `chart--step-${step}`;
    (step === 0) ? console.log("no matching step") : updateChart(step);
    
  } else {
    chart.className = "chart";
    currentSectionText.textContent = "0";
  }
}, {
  rootMargin: "-50% 0% -50% 0%",
  threshold: [0, 0.25, 0.5, 0.75, 1]
});

targets.forEach(t => observer.observe(t));
invalidation.then(() => observer.disconnect());
```

<style>
  /* Container for the whole scrolling experience */
.scroll-container {
  position: relative;
  margin: 1rem auto;
  font-family: var(--sans-serif);
}

/* This element now takes the full viewport height */
.scroll-info {
  position: sticky;
  height: 100vh;          /* Full viewport height */
  top: 0;                 /* Sticks to the top */
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 64px;
  transition: background-color 0.5s ease;
  background-color: var(--theme-background-alt);
}

/* Optional: test element modifications, if any */
.chart {
  /* height: 100vh;          Ensure inner test is also full height */
  background-color: var(--theme-background-alt);
}

.chart--step-1 {
  background-color: #4269d0;
}

.chart--step-2 {
  background-color: #efb118;
}

.chart--step-3 {
  background-color: #ff725c;
}

.chart--step-4 {
  background-color: #6cc5b0;
}

/* Each scroll section now covers the full height of the viewport */
.scroll-section {
  position: relative;
  height: 100vh;          /* Full viewport height */
  /* margin: 1rem 0; */
  display: flex;
  align-items: start;
  justify-content: center;
  border: solid 1px var(--theme-foreground-focus);
  background: color-mix(in srgb, var(--theme-foreground-focus) 5%, transparent);
  padding: 1rem;
  box-sizing: border-box;
}

/* Responsive chart display remains the same */
#chart-desktop,
#chart-tablet,
#chart-mobile {
  display: none;
}

/*desktop*/
@media (min-width: 750px) {
  #chart-desktop {
    display: flex;
    justify-content: end;
    width: 720px;
    height: 509px;
    margin: 0 auto;
  }
  .text-section {
    position: absolute;
    top: 20%;
    left: 50%;
    transform: translate(-75%, -50%);
    width: 100%;
    display: flex;
    justify-content: center;
    z-index: 1000;
  }
}

/*tablet*/
@media (min-width: 618px) and (max-width: 749px) {
  #chart-tablet {
    display: block;
    width: 557px;
    height: 509px;
    margin: 0 auto;
  }
  .text-section {
    position: absolute;
    top: 20%;
    left: 45%;
    transform: translate(-100%, -50%);
    width: 50%;
    display: flex;
    justify-content: center;
    pointer-events: none;
    z-index: 1000;
    color: #4b535d;
    letter-spacing: 0.36px;
  }
  .text-box {
    font-size: 15px;
  }
}

/*mobile*/
@media (max-width: 617px) {
  #chart-mobile {
    display: block;
    width: 320px;
    height: 509px;
    margin: 0 auto;
  }
  .text-section {
    position: absolute;
    top: 20%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 100%;
    display: flex;
    justify-content: center;
    pointer-events: none;
    z-index: 1000;
    color: #4b535d;
    letter-spacing: 0.36px;
  }
  .text-box {
    font-size: 15px !important;
  }
}

#chart-container {
  position: relative;
  z-index: 1;
  display: flex;
}

.text-box {
  background: white;
  padding: 20px;
  max-width: 300px;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  opacity: 0.93;
  position: relative;
  margin: 0 20px;
  stroke: #7f7f7f;
  font-family: "Lyon-RegularNo2" !important;
  color: #4b535d;
  letter-spacing: 0.36px;
  font-size: 16px;
}

.text-avocado { 
  background-color: #507a3a; 
  color: #ffffff; 
  font-family: "Lyon-RegularNo2" !important;
  letter-spacing: 0.36px;
  font-size: 16px;
}

.text-tariff { 
  background-color: #d1601d; 
  color: #ffffff; 
  font-family: "Lyon-RegularNo2" !important;
  letter-spacing: 0.36px;
  font-size: 16px;
}

.text-profit { 
  background-color: #92a83b; 
  color: #ffffff; 
  font-family: "Lyon-RegularNo2" !important;
  letter-spacing: 0.36px;
  font-size: 16px;
}

.text-consumer { 
  background-color: #4b535d; 
  color: #ffffff; 
  font-family: "Lyon-RegularNo2" !important;
  letter-spacing: 0.36px;
  font-size: 16px;
}

</style>
