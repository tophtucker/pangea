---
source: https://observablehq.com/@d3/density-contours
index: false
draft: true
---

<div style="color: grey; font: 13px/25.5px var(--sans-serif); text-transform: uppercase;"><h1 style="display: none;">Density contours</h1><a href="https://d3js.org/">D3</a> › <a href="/@d3/gallery">Gallery</a></div>

# Density contours

This chart shows the relationship between idle and eruption times for [Old Faithful](https://en.wikipedia.org/wiki/Old_Faithful).

```js echo
const chart = {
  // Specify the dimensions of the chart.
  const width = 928;
  const height = 600;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 30;
  const marginLeft = 40;

  // Create the horizontal and vertical scales.
  const x = d3.scaleLinear()
      .domain(d3.extent(faithful, d => d.waiting)).nice()
      .rangeRound([marginLeft, width - marginRight]);

  const y = d3.scaleLinear()
      .domain(d3.extent(faithful, d => d.eruptions)).nice()
      .rangeRound([height - marginBottom, marginTop]);

  // Compute the density contours.
  const contours = d3.contourDensity()
      .x(d => x(d.waiting))
      .y(d => y(d.eruptions))
      .size([width, height])
      .bandwidth(30)
      .thresholds(30)
    (faithful);

  // Create the SVG container.
  const svg = d3.create("svg")
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [0, 0, width, height])
      .attr("style", "max-width: 100%; height: auto;");

  // Append the axes.
  svg.append("g")
      .attr("transform", `translate(0,${height - marginBottom})`)
      .call(d3.axisBottom(x).tickSizeOuter(0))
      .call(g => g.select(".domain").remove())
      .call(g => g.select(".tick:last-of-type text").clone()
        .attr("y", -3)
        .attr("dy", null)
        .attr("font-weight", "bold")
        .text("Idle (min.)"));

  svg.append("g")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(y).tickSizeOuter(0))
    .call(g => g.select(".domain").remove())
    .call(g => g.select(".tick:last-of-type text").clone()
        .attr("x", 3)
        .attr("text-anchor", "start")
        .attr("font-weight", "bold")
        .text("Erupting (min.)"));

  // Append the contours.
  svg.append("g")
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-linejoin", "round")
    .selectAll()
    .data(contours)
    .join("path")
      .attr("stroke-width", (d, i) => i % 5 ? 0.25 : 1)
      .attr("d", d3.geoPath());

  // Append dots.
  svg.append("g")
      .attr("stroke", "white")
    .selectAll()
    .data(faithful)
    .join("circle")
      .attr("cx", d => x(d.waiting))
      .attr("cy", d => y(d.eruptions))
      .attr("r", 2);

  return svg.node();
}
```

```js echo
const faithful = FileAttachment("faithful.tsv").tsv({typed: true});
```

Or you could use [Observable Plot](https://observablehq.com/plot)’s [density mark](/plot/marks/density). See the complete [Plot density contours example](/@observablehq/plot-point-cloud-density?intent=fork).

```js echo
Plot.density(faithful, {x: "waiting", y: "eruptions"}).plot({inset: 20});
```