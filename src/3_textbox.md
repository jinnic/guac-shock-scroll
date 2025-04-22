---
title: 03. Text box
toc: false
---

```js
import * as d3 from "npm:d3";
```

<!-- Scroll container -->
<section class="scroll-container">
  <div class="scroll-info">
    <div id="chart-container" class="chart ">
      <!-- testing here -->
      <div class="test">
          <p></p>
      </div>
      <!-- chart svg here -->
      <div id="chart-mobile" ></div>
      <div id="chart-tablet" ></div>
      <div id="chart-desktop" class="card"></div>
    </div>
  </div>
</section>

```js
// 1) Fetch data using FileAttachment
const text = await FileAttachment("data/sections-text.csv").csv({ typed: true }); // Text Box data
const data = await FileAttachment("data/guac-price.csv").csv({ typed: true }); // Bar Chart data

// Process chart data
data.forEach(d => {
  d.cost    = +d.cost;
  d.tariff  = +d.tariff;
  d.profit  = +d.profit;
  d.section = +d.section;
  d.total   = +d.total;
});
```

```js
// 2) Append scroll sections to container
const container = d3.select(".scroll-container");

// Create and append scroll sections based on the CSV data
const sections = container.selectAll(".scroll-section")
  .data(text.sort((a,b) => a.section - b.section))
  .enter()
  .append("div")
    .attr("class", "scroll-section")
    .attr("data-step", d => d.section);

// Add the title to each section
sections.append("h1")
  .text(d => d.title);

// Add the text-section with proper data-group attribute
const textSections = sections.append("div")
  .attr("class", "text-section")
  .attr("data-group", d => d.group || `group${d.section}`); // Use group from data or create default

// Add the text-box with HTML content
textSections.append("div")
  .attr("class", "text-box")
  .html(d => d.text);

// 3) Set up chart dimensions and scales
const width  = 100, height = 300;
const margin = { top: 20, right: 20, bottom: 20, left: 20 };
const keys   = ["cost", "tariff", "profit"];
const color = d3.scaleOrdinal()
  .domain(keys)
  .range(["#507a3a", "#a6bf43", "#d1601d"]);

const x = d3.scaleBand()
  .domain(["bar"])
  .range([margin.left, width - margin.right])
  .padding(0.3);

const y = d3.scaleLinear()
  .range([height - margin.bottom, margin.top]);

const svg = d3.create("svg")
  .attr("viewBox", [0,0,width,height]);

document.getElementById("chart-desktop").appendChild(svg.node());

// 4) Create utility function for segments
function getSegments(sectionNum) {
  // Find matching section row
  let row = data.find(d => d.section === sectionNum);
  // But if not found (e.g. sectionNum === 0), use an all-zeros row
  if (!row) {
    row = { cost: 0, tariff: 0, profit: 0, total: 0 };
  }
  const series = d3.stack().keys(keys)([row]);
  return series.map(layer => ({
    key:   layer.key,
    y0:    layer[0][0],
    y1:    layer[0][1],
    value: row[layer.key],
    total: row.total
  }));
}

// 5) Set up transition settings
const DUR   = 800;
const EASE  = d3.easeCubicOut;
const DELAY = { segments: 0, values: 200, total: 400 };

// 6) Initialize chart groups
svg.append("g").attr("class", "bars");
svg.append("g").attr("class", "values");
svg.append("g").attr("class", "totals");

// 7) Define update function
function updateChart(sectionNumber) {
  const segs = getSegments(+sectionNumber);
  const maxY = d3.max(segs, d => d.y1);
  y.domain([0, maxY]).nice();

  // --- segments ---
  svg.select("g.bars").selectAll("rect.segment")
    .data(segs, d => d.key)
    .join(
      enter => enter.append("rect")
        .attr("class", "segment")
        .attr("x", x("bar"))
        .attr("width", x.bandwidth())
        .attr("y", y(0))
        .attr("height", 0)
        .attr("fill", d => color(d.key))
        .call(g => g.transition().duration(DUR).ease(EASE)
          .attr("y", d => y(d.y1))
          .attr("height", d => y(d.y0) - y(d.y1))
        ),

      update => update.transition().duration(DUR).ease(EASE)
        .attr("y", d => y(d.y1))
        .attr("height", d => y(d.y0) - y(d.y1)),

      exit => exit.transition().duration(DUR/2)
        .attr("y", y(0))
        .attr("height", 0)
        .remove()
    );

  // --- value labels ---
  svg.select("g.values").selectAll("text.value")
    .data(segs.filter(d => d.value > 0), d => d.key)
    .join(
      enter => enter.append("text")
        .attr("class", "value")
        .attr("x", x("bar") + x.bandwidth()/2)
        .attr("y", y(0))
        .attr("fill", "white")
        .attr("font-size", "10px")
        .attr("text-anchor", "middle")
        .attr("opacity", 0)
        .text(d => `$${d.value.toFixed(2)}`)
        .call(g => g.transition().delay(DELAY.values).duration(DUR).ease(EASE)
          .attr("y", d => (y(d.y0) + y(d.y1))/2 + 2)
          .attr("opacity", 1)
        ),

      update => update.transition().delay(DELAY.values).duration(DUR).ease(EASE)
        .attr("y", d => (y(d.y0) + y(d.y1))/2 + 2)
        .attr("opacity", 1)
        .text(d => `$${d.value.toFixed(2)}`),

      exit => exit.transition().duration(DUR/2)
        .attr("opacity", 0)
        .remove()
    );

  // --- total label (above last segment) ---
  const topSeg = segs[segs.length-1];
  const totalArr = topSeg.total > 0 ? [topSeg] : [];
  svg.select("g.totals").selectAll("text.total")
    .data(totalArr, d => d.key)
    .join(
      enter => enter.append("text")
        .attr("class", "total")
        .attr("x", x("bar") + x.bandwidth()/2)
        .attr("y", y(0))
        .attr("text-anchor", "middle")
        .attr("font-weight", "bold")
        .attr("opacity", 0)
        .text(d => `$${d.total.toFixed(2)}`)
        .call(g => g.transition().delay(DELAY.total).duration(DUR).ease(EASE)
          .attr("y", y(topSeg.y1) - 8)
          .attr("opacity", 1)
        ),

      update => update.transition().delay(DELAY.total).duration(DUR).ease(EASE)
        .attr("y", y(topSeg.y1) - 8)
        .attr("opacity", 1)
        .text(d => `$${d.total.toFixed(2)}`),

      exit => exit.remove()
    );
}

// 8) Set up the intersection observer AFTER sections are created
// (This was likely the issue - observer was set up before sections existed)
const info = document.querySelector(".test");
const secs = document.querySelectorAll(".scroll-section");

const obs = new IntersectionObserver((ents) => {
  let best = ents.filter(e => e.isIntersecting)
                 .sort((a,b) => b.intersectionRatio - a.intersectionRatio)[0];
  if (best) {
    const step = +best.target.dataset.step;
    info.textContent = step;
    info.className = `test test--step-${step}`;
    updateChart(step);
  } else {
    info.textContent = "0";
    info.className = "test";
    updateChart(0);
  }
}, {
  rootMargin: "-50% 0% -50% 0%",
  threshold: [0, 0.25, 0.5, 0.75, 1]
});

// 9) Start observing sections and initialize chart
secs.forEach(s => obs.observe(s));
updateChart(1); // Initialize the chart with section 1 data

invalidation.then(() => obs.disconnect());
```

<!-- ```js
(async function(){
  // ————————————————
  // 1) Load both CSVs
  // ————————————————
  const text = await FileAttachment("data/sections-text.csv")
    .csv({typed:true});
  const data = await FileAttachment("data/guac-price.csv")
    .csv({typed:true});

  // ————————————————
  // 2) Build the scroll sections
  // ————————————————
  const container = d3.select(".scroll-container");
  const secs = container.selectAll(".scroll-section")
    .data(text, d=>d.section)
    .enter().append("div")
      .attr("class","scroll-section")
      .attr("data-step", d=>d.section);

  secs.append("h1").text(d=>d.title);
  secs.append("div")
      .attr("class","text-section")
      .attr("data-group", d=>d.group || `group${d.section}`)
    .append("div")
      .attr("class","text-box")
      .html(d=>d.text);

  // ————————————————
  // 3) Pre‑process the guac data & create the SVG + groups
  // ————————————————
  data.forEach(d => {
    d.cost=+d.cost; d.tariff=+d.tariff; 
    d.profit=+d.profit; d.section=+d.section; 
    d.total=+d.total;
  });

  const width=100, height=300;
  const margin={top:20,right:20,bottom:20,left:20};
  const keys=["cost","tariff","profit"];
  const color = d3.scaleOrdinal().domain(keys)
    .range(["#507a3a","#a6bf43","#d1601d"]);
  const x = d3.scaleBand()
    .domain(["bar"])
    .range([margin.left, width-margin.right])
    .padding(0.3);
  const y = d3.scaleLinear()
    .range([height-margin.bottom, margin.top]);

  const svg = d3.create("svg")
    .attr("viewBox",[0,0,width,height]);
  document.getElementById("chart-desktop")
    .appendChild(svg.node());

  svg.append("g").attr("class","bars");
  svg.append("g").attr("class","values");
  svg.append("g").attr("class","totals");

  function getSegments(sec){
    let row = data.find(d=>d.section===sec)
      || {cost:0,tariff:0,profit:0,total:0};
    return d3.stack().keys(keys)([row])
      .map(layer=>({
        key: layer.key,
        y0: layer[0][0],
        y1: layer[0][1],
        value: row[layer.key],
        total: row.total
      }));
  }

  const DUR=800, EASE=d3.easeCubicOut;
  const DELAY={segments:0,values:200,total:400};

  function updateChart(sectionNumber){
    const segs = getSegments(sectionNumber);
    y.domain([0,d3.max(segs,d=>d.y1)]).nice();

    // bars
    svg.select("g.bars").selectAll("rect.segment")
      .data(segs, d=>d.key)
      .join(
        enter => enter.append("rect")
          .attr("class","segment")
          .attr("x", x("bar"))
          .attr("width", x.bandwidth())
          .attr("y", y(0))
          .attr("height",0)
          .attr("fill", d=>color(d.key))
          .call(g=>g.transition().duration(DUR).ease(EASE)
            .attr("y", d=>y(d.y1))
            .attr("height", d=>y(d.y0)-y(d.y1))
          ),
        update => update.transition().duration(DUR).ease(EASE)
          .attr("y", d=>y(d.y1))
          .attr("height", d=>y(d.y0)-y(d.y1)),
        exit => exit.transition().duration(DUR/2)
          .attr("y", y(0))
          .attr("height", 0)
          .remove()
      );

    // values
    svg.select("g.values").selectAll("text.value")
      .data(segs.filter(d=>d.value>0), d=>d.key)
      .join(
        enter => enter.append("text")
          .attr("class","value")
          .attr("x", x("bar")+x.bandwidth()/2)
          .attr("y", y(0))
          .attr("fill","white")
          .attr("font-size","10px")
          .attr("text-anchor","middle")
          .attr("opacity",0)
          .text(d=>`$${d.value.toFixed(2)}`)
          .call(g=>g.transition().delay(DELAY.values).duration(DUR).ease(EASE)
            .attr("y",(d=> (y(d.y0)+y(d.y1))/2 + 2))
            .attr("opacity",1)
          ),
        update => update.transition().delay(DELAY.values).duration(DUR).ease(EASE)
          .attr("y", d=> (y(d.y0)+y(d.y1))/2 + 2)
          .attr("opacity",1)
          .text(d=>`$${d.value.toFixed(2)}`),
        exit => exit.transition().duration(DUR/2)
          .attr("opacity",0)
          .remove()
      );

    // total
    const top = segs[segs.length-1];
    const totalArr = top.total>0 ? [top] : [];
    svg.select("g.totals").selectAll("text.total")
      .data(totalArr, d=>d.key)
      .join(
        enter => enter.append("text")
          .attr("class","total")
          .attr("x", x("bar")+x.bandwidth()/2)
          .attr("y", y(0))
          .attr("text-anchor","middle")
          .attr("font-weight","bold")
          .attr("opacity",0)
          .text(d=>`$${d.total.toFixed(2)}`)
          .call(g=>g.transition().delay(DELAY.total).duration(DUR).ease(EASE)
            .attr("y", y(top.y1)-8)
            .attr("opacity",1)
          ),
        update => update.transition().delay(DELAY.total).duration(DUR).ease(EASE)
          .attr("y", y(top.y1)-8)
          .attr("opacity",1)
          .text(d=>`$${d.total.toFixed(2)}`),
        exit => exit.remove()
      );
  }

  // ————————————————
  // 4) Observer + initial seed
  // ————————————————
  const info = document.querySelector(".test");
  const observer = new IntersectionObserver((entries) => {
    const best = entries
      .filter(e=>e.isIntersecting)
      .sort((a,b)=>b.intersectionRatio - a.intersectionRatio)[0];
    const step = best ? +best.target.dataset.step : 0;
    info.textContent = step;
    info.className   = step ? `test test--step-${step}` : "test";
    updateChart(step);
  }, {
    rootMargin: "-50% 0% -50% 0%",
    threshold : [0,0.25,0.5,0.75,1]
  });

  d3.selectAll(".scroll-section").nodes()
    .forEach(node => observer.observe(node));

  // seed the chart so you see SECTION 1 before any scroll
  updateChart(1);
})();
``` -->



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
.test {
  position: absolute;
  left: 10px;
  background-color: var(--theme-background-alt);
}

.test--step-1 {
  background-color: #4269d0;
}

.test--step-2 {
  background-color: #efb118;
}

.test--step-3 {
  background-color: #ff725c;
}

.test--step-4 {
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
