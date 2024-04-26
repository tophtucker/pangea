---
source: https://observablehq.com/@d3/bar-chart-race
index: true
---

# Bar chart race

This chart animates the value (in $M) of the top global brands from 2000 to 2019. Color indicates sector. See [the explainer](https://observablehq.com/d/e9e3929cf7c50b45) for more. Data: [Interbrand](https://www.interbrand.com/best-brands/)

```js
const replay = view(html`<button>Replay</button>`);
```

```js echo
replay;

const svg = d3
  .create("svg")
  .attr("viewBox", [0, 0, width, height])
  .attr("width", width)
  .attr("height", height)
  .attr("style", "max-width: 100%; height: auto;");

const updateBars = bars(svg);
const updateAxis = axis(svg);
const updateLabels = labels(svg);
const updateTicker = ticker(svg);

const chart = display(svg.node());
```

```js echo
(async () => {
  for (const keyframe of keyframes) {
    const transition = svg.transition().duration(duration).ease(d3.easeLinear);

    // Extract the top bar’s value.
    x.domain([0, keyframe[1][0].value]);

    updateAxis(keyframe, transition);
    updateBars(keyframe, transition);
    updateLabels(keyframe, transition);
    updateTicker(keyframe, transition);

    invalidation.then(() => svg.interrupt());
    await transition.end();
  }
})();
```

```js echo
const data = FileAttachment("../data/category-brands.csv").csv({typed: true});
```

```js echo
const duration = 250;
const n = 12;
const names = new Set(data.map((d) => d.name));
const datevalues = Array.from(
  d3.rollup(
    data,
    ([d]) => d.value,
    (d) => +d.date,
    (d) => d.name
  )
)
  .map(([date, data]) => [new Date(date), data])
  .sort(([a], [b]) => d3.ascending(a, b));
```

```js echo
function rank(value) {
  const data = Array.from(names, (name) => ({name, value: value(name)}));
  data.sort((a, b) => d3.descending(a.value, b.value));
  for (let i = 0; i < data.length; ++i) data[i].rank = Math.min(n, i);
  return data;
}
```

```js echo
const k = 10;
const keyframes = [];
let ka, a, kb, b;
for ([[ka, a], [kb, b]] of d3.pairs(datevalues)) {
  for (let i = 0; i < k; ++i) {
    const t = i / k;
    keyframes.push([
      new Date(ka * (1 - t) + kb * t),
      rank((name) => (a.get(name) || 0) * (1 - t) + (b.get(name) || 0) * t)
    ]);
  }
}
keyframes.push([new Date(kb), rank((name) => b.get(name) || 0)]);
const nameframes = d3.groups(
  keyframes.flatMap(([, data]) => data),
  (d) => d.name
);

const prev = new Map(nameframes.flatMap(([, data]) => d3.pairs(data, (a, b) => [b, a])));
const next = new Map(nameframes.flatMap(([, data]) => d3.pairs(data)));
```

```js echo
function bars(svg) {
  let bar = svg.append("g").attr("fill-opacity", 0.6).selectAll("rect");

  return ([date, data], transition) =>
    (bar = bar
      .data(data.slice(0, n), (d) => d.name)
      .join(
        (enter) =>
          enter
            .append("rect")
            .attr("fill", color)
            .attr("height", y.bandwidth())
            .attr("x", x(0))
            .attr("y", (d) => y((prev.get(d) || d).rank))
            .attr("width", (d) => x((prev.get(d) || d).value) - x(0)),
        (update) => update,
        (exit) =>
          exit
            .transition(transition)
            .remove()
            .attr("y", (d) => y((next.get(d) || d).rank))
            .attr("width", (d) => x((next.get(d) || d).value) - x(0))
      )
      .call((bar) =>
        bar
          .transition(transition)
          .attr("y", (d) => y(d.rank))
          .attr("width", (d) => x(d.value) - x(0))
      ));
}
```

```js echo
function labels(svg) {
  let label = svg
    .append("g")
    .style("font", "bold 12px var(--sans-serif)")
    .style("font-variant-numeric", "tabular-nums")
    .attr("text-anchor", "end")
    .attr("fill", "currentColor")
    .selectAll("text");

  return ([date, data], transition) =>
    (label = label
      .data(data.slice(0, n), (d) => d.name)
      .join(
        (enter) =>
          enter
            .append("text")
            .attr("transform", (d) => `translate(${x((prev.get(d) || d).value)},${y((prev.get(d) || d).rank)})`)
            .attr("y", y.bandwidth() / 2)
            .attr("x", -6)
            .attr("dy", "-0.25em")
            .text((d) => d.name)
            .call((text) =>
              text
                .append("tspan")
                .attr("fill-opacity", 0.7)
                .attr("font-weight", "normal")
                .attr("x", -6)
                .attr("dy", "1.15em")
            ),
        (update) => update,
        (exit) =>
          exit
            .transition(transition)
            .remove()
            .attr("transform", (d) => `translate(${x((next.get(d) || d).value)},${y((next.get(d) || d).rank)})`)
            .call((g) => g.select("tspan").tween("text", (d) => textTween(d.value, (next.get(d) || d).value)))
      )
      .call((bar) =>
        bar
          .transition(transition)
          .attr("transform", (d) => `translate(${x(d.value)},${y(d.rank)})`)
          .call((g) => g.select("tspan").tween("text", (d) => textTween((prev.get(d) || d).value, d.value)))
      ));
}
```

```js echo
function textTween(a, b) {
  const i = d3.interpolateNumber(a, b);
  return function (t) {
    this.textContent = formatNumber(i(t));
  };
}
```

```js echo
const formatNumber = d3.format(",d");
const tickFormat = undefined; // override as desired
```

```js echo
function axis(svg) {
  const g = svg.append("g").attr("transform", `translate(0,${marginTop})`);

  const axis = d3
    .axisTop(x)
    .ticks(width / 160, tickFormat)
    .tickSizeOuter(0)
    .tickSizeInner(-barSize * (n + y.padding()));

  return (_, transition) => {
    g.transition(transition).call(axis);
    g.select(".tick:first-of-type text").remove();
    g.selectAll(".tick:not(:first-of-type) line").attr("stroke", "white");
    g.select(".domain").remove();
  };
}
```

```js echo
function ticker(svg) {
  const now = svg
    .append("text")
    .style("font", `bold ${barSize}px var(--sans-serif)`)
    .style("font-variant-numeric", "tabular-nums")
    .attr("text-anchor", "end")
    .attr("x", width - 6)
    .attr("y", marginTop + barSize * (n - 0.45))
    .attr("dy", "0.32em")
    .attr("fill", "currentColor")
    .text(formatDate(keyframes[0][0]));

  return ([date], transition) => {
    transition.end().then(() => now.text(formatDate(date)));
  };
}
```

```js echo
let color;
const scale = d3.scaleOrdinal(d3.schemeObservable10);
if (data.some((d) => d.category !== undefined)) {
  const categoryByName = new Map(data.map((d) => [d.name, d.category]));
  scale.domain(categoryByName.values());
  color = (d) => scale(categoryByName.get(d.name));
} else color = (d) => scale(d.name);

const formatDate = d3.utcFormat("%Y");

const x = d3.scaleLinear([0, 1], [marginLeft, width - marginRight]);
const y = d3
  .scaleBand()
  .domain(d3.range(n + 1))
  .rangeRound([marginTop, marginTop + barSize * (n + 1 + 0.1)])
  .padding(0.1);
```

```js echo
const barSize = 48;
const marginTop = 16;
const marginRight = 6;
const marginBottom = 6;
const marginLeft = 0;
const height = marginTop + barSize * n + marginBottom;
```