---
title: Test area - not working (code dump)
toc: false
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
// Convert text to numbers
data.forEach((d) => {
  d.cost = +d.cost;
  d.tariff = +d.tariff;
  d.profit = +d.profit;
  d.section = +d.section;
  d.total = +d.total;
});

const width = 100,
  height = 300;
const margin = { top: 20, right: 20, bottom: 20, left: 20 };

// "profit", "cost", "tariff" : Keys for stacked bar chart
// const keys = data.columns.slice(1, 4);
const keys = ["cost", "tariff", "profit"];

// Color scale for stacked bar
const color = d3
  .scaleOrdinal()
  .domain(keys)
  .range(["#507a3a", "#a6bf43", "#d1601d"]);

// Create SVG only once
const svg = d3.create("svg").attr("viewBox", [0, 0, width, height]);

// Append the chart to a container element in the DOM as in original code
document.getElementById("chart-desktop").appendChild(svg.node());

// Function to update the chart based on the current section
function updateChart(sectionNumber) {
  const sectionNum = +sectionNumber;

  // Filter data to show only the section matching the current section
  const currentData = data.filter(d => d.section === sectionNum);
  console.log("currentData: ", data, currentData);

  // If no data matches, don't update currentData.length === 0
  if (!currentData) {
    console.warn("No data found for section", sectionNum);
    return;
  }

  // Set up the stack generator
  const stackGen = d3.stack().keys(keys).offset(d3.stackOffsetNone);
  const series = stackGen(currentData);

  // Get the maximum value for the y-scale 
  const maxValue = d3.max(series[series.length - 1], (d) => d[1]) || 0;

  // Clear previous chart content
  // svg.selectAll("*").remove();

  // Set up scales 
  const x = d3
    .scaleBand()
    .domain(currentData.map(d => d.section)) //[currentData[0].section])
    .range([margin.left, width - margin.right])
    .padding(0.2);

  const y = d3
    .scaleLinear()
    .domain([0, maxValue]).nice()
    .range([height - margin.bottom, margin.top]);


//   // 1) DRAW & ANIMATE THE BARS
// svg.selectAll("g.layer")
//   .data(series)
//   .join("g")
//     .attr("fill", d => color(d.key))
//   .selectAll("rect")
//   .data(d => d)
//   .join(
//     enter => enter
//       // start each new rect at height=0 (flat on the x‐axis)
//       .append("rect")
//         .attr("x", d => x(d.data.section))
//         .attr("width", x.bandwidth())
//         .attr("y", y(0))
//         .attr("height", 0)
//       .call(enter =>
//         enter.transition()
//           .duration(800)
//           .ease(d3.easeCubicOut)
//           // animate up to its stacked y1 & height
//           .attr("y", d => y(d[1]))
//           .attr("height", d => y(d[0]) - y(d[1]))
//       ),

//     update => update
//       // for updates, you may want to transition from old to new
//       .call(update =>
//         update.transition()
//           .duration(800)
//           .ease(d3.easeCubicOut)
//           .attr("x", d => x(d.data.section))
//           .attr("y", d => y(d[1]))
//           .attr("height", d => y(d[0]) - y(d[1]))
//       ),

//     exit => exit
//       .call(exit =>
//         exit.transition()
//           .duration(500)
//           .attr("y", y(0))
//           .attr("height", 0)
//           .remove()
//       )
//   );

// // 2) ANIMATE THE LABELS (optional)
// svg.selectAll("g.label")
//   .data(series)
//   .join("g")
//     .attr("class", "label")
//   .selectAll("text")
//   .data(layer =>
//     layer.map(item => ({
//       key: layer.key,
//       value: item.data[layer.key],
//       y0: item[0],
//       y1: item[1],
//       section: item.data.section
//     }))
//   )
//   .join(
//     enter => enter
//       .append("text")
//         .attr("x", d => x(d.section) + x.bandwidth()/2)
//         // start labels at the bottom of the bar
//         .attr("y", y(0))
//         .attr("opacity", 0)
//         .attr("text-anchor", "middle")
//         .attr("fill", "white")
//         .attr("font-size", "9px")
//         .attr("font-weight", "bold")
//         .text(d => `$${d.value.toFixed(2)}`)
//       .call(enter =>
//         enter.transition()
//           .delay(300)               // wait until bars have begun rising
//           .duration(500)
//           .attr("opacity", 1)
//           .attr("y", d => {
//             const mid = y(d.y0) - (y(d.y0) - y(d.y1))/2;
//             return mid + 5;
//           })
//       ),

//     update => update
//       .call(update =>
//         update.transition()
//           .duration(500)
//           .attr("x", d => x(d.section) + x.bandwidth()/2)
//           .attr("y", d => {
//             const mid = y(d.y0) - (y(d.y0) - y(d.y1))/2;
//             return mid + 5;
//           })
//       ),

//     exit => exit
//       .call(exit =>
//         exit.transition()
//           .duration(300)
//           .attr("opacity", 0)
//           .remove()
//       )
//   );
  // // Draw the stacked bars
  // svg.selectAll("g.layer")
  //   .data(series)
  //   .join("g")
  //     .attr("class", "layer")
  //     .attr("fill", (d) => color(d.key))
  //   .selectAll("rect")
  //   .data((d) => d)
  //   .join("rect")
  //     .attr("x", (d) => x(d.data.section))
  //     .attr("y", (d) => y(d[1]))
  //     .attr("width", x.bandwidth())
  //     .attr("height", (d) => y(d[0]) - y(d[1]))
  //   .transition()
  //     .duration(1000);

  // // Add labels inside each bar segment
  // svg.selectAll("g.label")
  //   .data(series)
  //   .join("g")
  //     .attr("class", "label")
  //   .selectAll("text")
  //   .data((d) => {
  //     // Add the key name to each data point
  //     return d.map((item) => ({
  //       key: d.key,
  //       value: item.data[d.key],
  //       y0: item[0],
  //       y1: item[1],
  //       section: item.data.section
  //     }));
  //   })
  //   .join("text")
  //     .attr("x", (d) => x(d.section)+x.bandwidth() / 2)
  //     .attr("y", (d) => {
  //       // Position in the middle of each segment
  //       const mid = y(d.y0) - (y(d.y0) - y(d.y1)) / 2;
  //       return mid + 5; // Adjust for text alignment
  //     })
  //     .attr("text-anchor", "middle")
  //     .attr("fill", "white") // White text for better contrast
  //     .attr("font-size", "9px")
  //     .attr("font-weight", "bold")
  //   .text((d) => {
  //     // Only show label if the segment is tall enough
  //     const height = y(d.y0) - y(d.y1);
  //     if (height < 20) return ""; // Skip small segments
  //     return `$${d.value.toFixed(2)}`;
  //   });

  // Total label on top
  svg.selectAll("text.total")
    .data(currentData)
    .join("text")
      .attr("class", "total")
      .attr("x", (d) => x(d.section) + x.bandwidth()/2)
      .attr("y", (d) => y(d.cost + d.tariff + d.profit) - 5)
      .attr("text-anchor", "middle")
    .text((d) => `${(d.cost + d.tariff + d.profit).toFixed(2)}`);
}

// Modify the intersection observer to update the chart when section changes
const currentSectionText = document.querySelector(".test");
const chart = document.querySelector(".chart");
const targets = document.querySelectorAll(".scroll-section");

const observer = new IntersectionObserver(
  (entries) => {
    let bestEntry = null;
    // Loop through all entries reported by the observer
    for (const entry of entries) {
      if (entry.isIntersecting) {
        if (
          !bestEntry ||
          entry.intersectionRatio > bestEntry.intersectionRatio
        ) {
          bestEntry = entry;
        }
      }
    }
    if (bestEntry) {
      // Update the info element with the data-step of the most visible element
      const currentStep = bestEntry.target.dataset.step;
      currentSectionText.textContent = currentStep;
      chart.className = `chart--step-${currentStep}`;
      console.log("Current Step: :", currentStep, bestEntry);
      // Update the chart based on the current section
      updateChart(currentStep);
    } else {
      // If no section is considered visible, reset the info element
      chart.className = "chart";
      currentSectionText.textContent = "0";
    }
  },
  {
    // Keep the same rootMargin as original
    rootMargin: "-50% 0% -50% 0%",
    // Use same thresholds for better detection of which section is most visible
    threshold: [0, 0.25, 0.5, 0.75, 1],
  }
);

// Start observing each target section
targets.forEach((target) => observer.observe(target));

// Initialize the chart with the first section
updateChart(1);

// When the external promise 'invalidation' resolves, disconnect the observer
invalidation.then(() => observer.disconnect());
```

```js
// OLD VERSION
// const info = document.querySelector(".test");
// const targets = document.querySelectorAll(".scroll-section");
// // console.log("this is targets: ", targets);
// const observer = new IntersectionObserver(
//   (entries) => {
//     for (const target of Array.from(targets).reverse()) {
//       const rect = target.getBoundingClientRect();
//       if (rect.top < innerHeight / 2) {
//         info.textContent = target.dataset.step;
//         info.className = `test--step-${target.dataset.step}`;
//         return;
//       }
//     }
//     info.className = "test";
//     info.textContent = "0";
//   },
//   {
//     rootMargin: "-50% 0% -50% 0%",
//   }
// );

// for (const target of targets) observer.observe(target);

// invalidation.then(() => observer.disconnect());
```

````js
// const info = document.querySelector(".test");
// const targets = document.querySelectorAll(".scroll-section");

// const observer = new IntersectionObserver(
//   (entries) => {
//     let bestEntry = null;
//     // Loop through all entries reported by the observer.
//     for (const entry of entries) {
//       // Only consider entries that are intersecting (visible in the viewport).
//       if (entry.isIntersecting) {
//         // If we haven't chosen an entry yet, or this one has a higher intersectionRatio, select it.
//         if (
//           !bestEntry ||
//           entry.intersectionRatio > bestEntry.intersectionRatio
//         ) {
//           bestEntry = entry;
//         }
//       }
//     }
//     if (bestEntry) {
//       // Update the info element with the data-step of the most visible element.
//       const currentStep = bestEntry.target.dataset.step;
//       info.textContent = bestEntry.target.dataset.step;
//       info.className = `test--step-${bestEntry.target.dataset.step}`;
//       // Update the chart based on the current section
//       updateChart(currentStep);
//     } else {
//       // If no section is considered visible, reset the info element.
//       info.textContent = "0";
//       info.className = "test";
//     }
//   },
//   {
//     // Adjusting the rootMargin lets us focus on a specific region (here the center) of the viewport.
//     rootMargin: "-50% 0% -50% 0%",
//     // Adding thresholds so the callback is triggered at different levels of visibility.
//     threshold: [0, 0.25, 0.5, 0.75, 1],
//   }
// );

// // Start observing each target section.
// targets.forEach((target) => observer.observe(target));
// // Initialize the chart with the first section
// updateChart(0);

// // When the external promise 'invalidation' resolves, disconnect the observer.
// invalidation.then(() => observer.disconnect());
// ```

// ```js
// // Conver text to number
// data.forEach((d) => {
//     d.cost = +d.cost;
//     d.tariff = +d.tariff;
//     d.profit = +d.profit;
//   }

// console.log("new data: ",data);
````

<!--
```js
// const keys = ["cost", "tariff", "profit"];
const keys = data.columns.slice(1, 4);
console.log(keys)
const stackGen = d3.stack().keys(keys).offset(d3.stackOffsetNone);
const series = stackGen(data);
const maxValue = d3.max(series[series.length - 1], (d) => d[1]) || 0;

const width = 500,
  height = 300;
const margin = { top: 20, right: 20, bottom: 20, left: 20 };
console.log("This is keys: ", Object.keys(data[0]));
const x = d3
  .scaleBand()
  .domain(data.map((d) => d.section))
  .range([margin.left, width - margin.right])
  .padding(0.2);

const y = d3
  .scaleLinear()
  .domain([0, maxValue])
  .nice()
  .range([height - margin.bottom, margin.top]);

const color = d3
  .scaleOrdinal()
  .domain(keys)
  .range(["#6da48f", "#d1601d", "#92a83b"]);

const svg = d3.create("svg").attr("viewBox", [0, 0, width, height]);

svg
  .selectAll("g.layer")
  .data(series)
  .join("g")
  .attr("class", "layer")
  .attr("fill", (d) => color(d.key))
  .selectAll("rect")
  .data((d) => d)
  .join("rect")
  .attr("x", (d) => x(d.data.section))
  .attr("y", (d) => y(d[1]))
  .attr("width", x.bandwidth())
  .attr("height", (d) => y(d[0]) - y(d[1]));

console.log("svg = ", svg);
// Append the chart to a container element in the DOM:
document.getElementById("chart-desktop").appendChild(svg.node());
```
 -->

<!--
```js
// D3 Avocado Tariff Visualization with Intersection Observer
document.addEventListener("DOMContentLoaded", function () {
  console.log("hello event triggered");
  // --- 1. DEFINE DATA FOR EACH SCENARIO ---
  const scenarios = [
    {
      id: "groupA",
      title: "Before tariffs",
      components: [
        { name: "Import price", value: 1.0, color: "#507a3a", label: "$1.00" },
        { name: "Profit", value: 0.5, color: "#92a83b", label: "$0.50" },
      ],
    },
    {
      id: "groupB",
      title: "With 25% tariff (passed to consumer)",
      components: [
        { name: "Import price", value: 1.0, color: "#507a3a", label: "$1.00" },
        { name: "Tariff", value: 0.25, color: "#d1601d", label: "$0.25" },
        { name: "Profit", value: 0.5, color: "#92a83b", label: "$0.50" },
      ],
    },
    {
      id: "groupC",
      title: "With 25% tariff (absorbed by company)",
      components: [
        { name: "Import price", value: 1.0, color: "#507a3a", label: "$1.00" },
        { name: "Tariff", value: 0.25, color: "#d1601d", label: "$0.25" },
        { name: "Profit", value: 0.25, color: "#92a83b", label: "$0.25" },
      ],
    },
    {
      id: "groupD",
      title: "With domestic supplier (higher price)",
      components: [
        {
          name: "Domestic price",
          value: 1.4,
          color: "#507a3a",
          label: "$1.40",
        },
        { name: "Profit", value: 0.5, color: "#92a83b", label: "$0.50" },
      ],
    },
  ];

  // Calculate total price for each scenario
  scenarios.forEach((scenario) => {
    scenario.totalPrice = scenario.components.reduce(
      (sum, component) => sum + component.value,
      0
    );
  });

  // --- 2. SET UP SVG CONTAINERS ---
  const chartMobile = document.getElementById("chart-mobile");
  const chartTablet = document.getElementById("chart-tablet");
  const chartDesktop = document.getElementById("chart-desktop");

  // Clear any existing content
  chartMobile.innerHTML = "";
  chartTablet.innerHTML = "";
  chartDesktop.innerHTML = "";

  // Create SVGs for different screen sizes
  const svgMobile = d3
    .select(chartMobile)
    .append("svg")
    .attr("width", 320)
    .attr("height", 509)
    .attr("viewBox", "0 0 320 509");

  const svgTablet = d3
    .select(chartTablet)
    .append("svg")
    .attr("width", 557)
    .attr("height", 509)
    .attr("viewBox", "0 0 557 509");

  const svgDesktop = d3
    .select(chartDesktop)
    .append("svg")
    .attr("width", 720)
    .attr("height", 509)
    .attr("viewBox", "0 0 720 509");

  console.log(chartDesktop);

  // --- 3. CREATE DRAW CHART FUNCTION ---
  function drawChart(scenarioIndex) {
    const scenario = scenarios[scenarioIndex];

    // Draw on all three SVGs (mobile, tablet, desktop)
    [
      { svg: svgMobile, width: 320 },
      { svg: svgTablet, width: 557 },
      { svg: svgDesktop, width: 720 },
    ].forEach((config) => {
      const { svg, width } = config;

      // Clear previous content
      svg.selectAll("*").remove();

      // Define dimensions based on SVG size
      const margin = {
        top: 120,
        right: width * 0.1,
        bottom: 50,
        left: width * 0.15,
      };

      const chartWidth = width - margin.left - margin.right;
      const height = 400;

      // Create scales
      const yScale = d3
        .scaleLinear()
        .domain([0, 2.0]) // Maximum price is $2
        .range([height - margin.bottom, margin.top]);

      const xScale = d3
        .scaleBand()
        .domain(["Price"])
        .range([margin.left, width - margin.right])
        .padding(0.5);

      // Create group for the visualization
      const g = svg.append("g");

      // Draw axes
      const yAxis = d3
        .axisLeft(yScale)
        .tickFormat((d) => `$${d.toFixed(2)}`)
        .ticks(5);

      g.append("g")
        .attr("transform", `translate(${margin.left},0)`)
        .call(yAxis)
        .attr("class", "y-axis")
        .selectAll("text")
        .style("font-size", width < 400 ? "10px" : "12px")
        .style("font-family", "Lyon-RegularNo2, serif");

      // Draw the stacked bars
      const barWidth = xScale.bandwidth();
      let yPosition = height - margin.bottom;

      // Create a group for the bar segments
      const barGroup = g.append("g").attr("id", `${scenario.id}_barstacked`);

      // Create a group for labels
      const labelGroup = g.append("g").attr("id", "labels");

      // Draw each component
      scenario.components.forEach((component, i) => {
        const barHeight = height - margin.bottom - yScale(component.value);

        // Draw segment
        barGroup
          .append("rect")
          .attr("x", xScale("Price"))
          .attr("y", yPosition - barHeight)
          .attr("width", barWidth)
          .attr("height", 0) // Start with height 0 for animation
          .attr("fill", component.color)
          .attr("class", "bar-segment")
          .transition()
          .duration(800)
          .attr("height", barHeight); // Animate to full height

        // Add label for this segment (for larger screens)
        if (width >= 400) {
          labelGroup
            .append("text")
            .attr(
              "id",
              `${scenario.id}_${component.name
                .replace(/\s+/g, "_")
                .toLowerCase()}_label`
            )
            .attr("x", xScale("Price") + barWidth / 2)
            .attr("y", yPosition - barHeight / 2)
            .attr("text-anchor", "middle")
            .attr("dominant-baseline", "middle")
            .text(component.name)
            .style("font-size", width < 500 ? "10px" : "12px")
            .style("font-weight", "bold")
            .style("fill", "white")
            .style("font-family", "Lyon-RegularNo2, serif")
            .style("opacity", 0)
            .transition()
            .duration(800)
            .delay(400)
            .style("opacity", 1);
        }

        // Add price label
        labelGroup
          .append("text")
          .attr(
            "id",
            `${scenario.id}_${component.name
              .replace(/\s+/g, "_")
              .toLowerCase()}_price_label`
          )
          .attr("x", xScale("Price") + barWidth + 10)
          .attr("y", yPosition - barHeight / 2)
          .attr("text-anchor", "start")
          .attr("dominant-baseline", "middle")
          .text(component.label)
          .style("font-size", width < 500 ? "10px" : "12px")
          .style("font-weight", "bold")
          .style("font-family", "Lyon-RegularNo2, serif")
          .style("opacity", 0)
          .transition()
          .duration(800)
          .delay(600)
          .style("opacity", 1);

        yPosition -= barHeight;
      });

      // Add total price marker and label
      g.append("rect")
        .attr("x", xScale("Price"))
        .attr("y", yScale(scenario.totalPrice) - 15)
        .attr("width", barWidth)
        .attr("height", 2)
        .attr("fill", "#4b535d");

      labelGroup
        .append("text")
        .attr("id", `${scenario.id}_total_price_label`)
        .attr("x", xScale("Price") + barWidth + 10)
        .attr("y", yScale(scenario.totalPrice) - 5)
        .attr("text-anchor", "start")
        .text(`Total: $${scenario.totalPrice.toFixed(2)}`)
        .style("font-size", width < 500 ? "12px" : "14px")
        .style("font-weight", "bold")
        .style("fill", "#4b535d")
        .style("font-family", "Lyon-RegularNo2, serif")
        .style("opacity", 0)
        .transition()
        .duration(800)
        .delay(800)
        .style("opacity", 1);

      // Add scenario title
      g.append("text")
        .attr("x", width / 2)
        .attr("y", margin.top / 2)
        .attr("text-anchor", "middle")
        .text(scenario.title)
        .style("font-size", width < 500 ? "14px" : "18px")
        .style("font-weight", "bold")
        .style("fill", "#4b535d")
        .style("font-family", "Lyon-RegularNo2, serif")
        .style("opacity", 0)
        .transition()
        .duration(800)
        .style("opacity", 1);

      // Add a stylized avocado icon (optional)
      const iconSize = width < 400 ? 30 : 40;
      g.append("text")
        .attr("x", width - margin.right - iconSize)
        .attr("y", margin.top / 2)
        .attr("text-anchor", "end")
        .text("🥑")
        .style("font-size", `${iconSize}px`);
    });
  }

  // --- 4. CONNECT D3 TO INTERSECTION OBSERVER ---
  const info = document.querySelector(".test");
  const scrollSections = document.querySelectorAll(".scroll-section");
  let currentStep = 0;

  // Create a new IntersectionObserver
  const observer = new IntersectionObserver(
    (entries) => {
      // Check each section in reverse order (to get the topmost visible section)
      for (const section of Array.from(scrollSections).reverse()) {
        const rect = section.getBoundingClientRect();

        // If this section is in the middle of the viewport
        if (rect.top < window.innerHeight / 2) {
          const step = parseInt(section.dataset.step);

          // Only update if the step has changed
          if (step !== currentStep) {
            console.log(`Updating to step ${step}`);
            currentStep = step;

            // Update info display (for debugging)
            info.textContent = step;
            info.className = `test--step-${step}`;

            // Update D3 visualization (step is 1-based, array is 0-based)
            drawChart(step - 1);
          }
          return;
        }
      }

      // If no section is visible, reset to step 0
      info.className = "test";
      info.textContent = "0";
    },
    {
      rootMargin: "-50% 0% -50% 0%", // Only consider middle 50% of viewport
    }
  );

  // Observe all scroll sections
  for (const section of scrollSections) {
    observer.observe(section);
  }

  // Initial chart render with first scenario
  drawChart(0);

  // Window resize handler for responsive behavior
  window.addEventListener("resize", () => {
    // Redraw current chart when window is resized
    drawChart(currentStep - 1);
  });

  // Clean up observer when page is unloaded
  window.addEventListener("beforeunload", () => {
    observer.disconnect();
  });
});
```
-->
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
