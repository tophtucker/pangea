---
source: https://observablehq.com/@d3/hertzsprung-russell-diagram
index: true
---

# Hertzsprung–Russell diagram

An [HR diagram](https://en.wikipedia.org/wiki/Hertzsprung–Russell_diagram) plots the relationship between stars’ absolute magnitudes (brighter going up) and temperatures (warmer going left). For the large set of stars below, it effectively shows how [stars age over time](https://en.wikipedia.org/wiki/Stellar_evolution). Data: [Hipparcos](http://cdsarc.u-strasbg.fr/viz-bin/cat/I/311), [Gliese](http://cdsarc.u-strasbg.fr/viz-bin/cat/V/70A)

```js echo
// Declare the chart dimensions and margins.
const width = 928;
const height = Math.round(width * 1.2);
const marginTop = 40;
const marginRight = 40;
const marginBottom = 40;
const marginLeft = 40;

// Ceate the scales.
const x = d3.scaleLinear([-0.39, 2.19], [marginLeft, width - marginRight]);
const y = d3.scaleLinear([-7, 19], [marginTop, height - marginBottom]);
const z = bv2rgb;

// Create the SVG container.
const svg = d3
  .create("svg")
  .attr("width", width + 28)
  .attr("height", height)
  .attr("viewBox", [-14, 0, width + 28, height])
  .attr("style", "max-width: calc(100% + 28px); height: auto;")
  .style("margin", "0 -14px")
  .style("background", "#000")
  .style("color", "#fff")
  .style("display", "block")
  .attr("fill", "currentColor")
  .attr("font-family", "sans-serif")
  .attr("font-size", 10);

// Create a small (sub pixel) rectangle for each star.
svg
  .append("g")
  .selectAll("rect")
  .data(data)
  .join("rect")
  .attr("x", (d) => x(d.color))
  .attr("y", (d) => y(d.absolute_magnitude))
  .attr("fill", (d) => z(d.color))
  .attr("width", 0.75)
  .attr("height", 0.75);

// Create the axes.
svg
  .append("g")
  .attr("transform", `translate(${marginLeft},0)`)
  .call(
    d3.axisLeft(
      d3.scaleLog(
        y.domain().map((m) => Math.pow(10, 4.83 - m)),
        y.range()
      )
    )
  );

svg
  .append("g")
  .attr("transform", `translate(${width - marginRight},0)`)
  .call(d3.axisRight(y).ticks(null, "+"));

svg
  .append("g")
  .attr("transform", `translate(0,${marginTop})`)
  .call(d3.axisTop(d3.scaleLinear(x.domain().map(temperature), x.range())));

svg
  .append("g")
  .attr("transform", `translate(0,${height - marginBottom})`)
  .call(d3.axisBottom(x).ticks(null, "+f"));

svg.selectAll(".domain").remove();

svg
  .append("text")
  .attr("dy", 12)
  .attr("text-anchor", "middle")
  .attr("transform", `translate(${marginLeft},${(marginTop + height - marginBottom) / 2}) rotate(-90)`)
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("← darker\xa0"))
  .call((text) => text.append("tspan").attr("font-weight", "bold").text("\xa0Luminosity L☉\xa0"))
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("\xa0brighter →"));

svg
  .append("text")
  .attr("dy", -6)
  .attr("text-anchor", "middle")
  .attr("transform", `translate(${width - marginRight},${(marginTop + height - marginBottom) / 2}) rotate(-90)`)
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("← darker\xa0"))
  .call((text) => text.append("tspan").attr("font-weight", "bold").text("\xa0Absolute magnitude M\xa0"))
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("\xa0brighter →"));

svg
  .append("text")
  .attr("x", (marginLeft + width - marginRight) / 2)
  .attr("y", marginTop)
  .attr("dy", 12)
  .attr("text-anchor", "middle")
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("← hotter\xa0"))
  .call((text) => text.append("tspan").attr("font-weight", "bold").text("\xa0Temperature K\xa0"))
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("\xa0colder →"));

svg
  .append("text")
  .attr("x", (marginLeft + width - marginRight) / 2)
  .attr("y", height - marginBottom)
  .attr("dy", -6)
  .attr("text-anchor", "middle")
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("← blue\xa0"))
  .call((text) => text.append("tspan").attr("font-weight", "bold").text("\xa0Color B-V\xa0"))
  .call((text) => text.append("tspan").attr("fill-opacity", 0.8).text("\xa0red →"));

display(svg.node());
```

```js echo
const data = FileAttachment("../data/catalog.csv").csv({typed: true});
```

See [this helper notebook](https://observablehq.com/d/e697f4807166cbd9) for the source data.

```js echo
function bv2rgb(bv) {
  bv = Math.max(-0.4, Math.min(2, bv));
  let t;
  return `#${[
    bv < 0 ? ((t = (bv + 0.4) / 0.4), 0.61 + 0.11 * t + 0.1 * t * t) : bv < 0.4 ? ((t = bv / 0.4), 0.83 + 0.17 * t) : 1,
    bv < 0
      ? ((t = (bv + 0.4) / 0.4), 0.7 + 0.07 * t + 0.1 * t * t)
      : bv < 0.4
      ? ((t = bv / 0.4), 0.87 + 0.11 * t)
      : bv < 1.6
      ? ((t = (bv - 0.4) / 1.2), 0.98 - 0.16 * t)
      : ((t = (bv - 1.6) / 0.4), 0.82 - 0.5 * t * t),
    bv < 0.4
      ? 1
      : bv < 1.5
      ? ((t = (bv - 0.4) / 1.1), 1 - 0.47 * t + 0.1 * t * t)
      : bv < 1.94
      ? ((t = (bv - 1.5) / 0.44), 0.63 - 0.6 * t * t)
      : 0
  ]
    .map((t) =>
      Math.round(t * 255)
        .toString(16)
        .padStart(2, "0")
    )
    .join("")}`;
}
```

```js echo
function temperature(color) {
  return 4600 * (1 / (0.92 * color + 1.7) + 1 / (0.92 * color + 0.62));
}
```
