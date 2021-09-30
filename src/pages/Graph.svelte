<script>
  import { scaleLinear, scaleTime } from "d3-scale";
  import { extent, bisector } from "d3-array";
  import { line, curveMonotoneX } from "d3-shape";
  import { pointer } from "d3-selection";
  import { capitalizeFirstLetter } from "../Utils.svelte";

  import { dictionary, locale, _ } from "svelte-i18n";
  import { get } from "svelte/store";

  import Axis from "./graph/Axis.svelte";
  import GraphTooltip from "./graph/GraphTooltip.svelte";
  import Definition from "./Definition.svelte";

  export let data;
  export let cat1;
  export let cat2;
  export let unit;
  export let graphID;

  let width;
  let hovered = null;

  let name = {};
  name["cat1"] = cat1;
  name["cat2"] = cat2;

  let array = {};
  array["cat1"] = [];
  array["cat2"] = [];

  const height = 300;
  const padding = {
    top: 30,
    bottom: 40,
    right: 2,
    left: 2,
  };
  let graphLimits = {};
  let lineGenerator = {};
  let graphInfos = {};

  graphInfos[graphID] = {};
  graphInfos[graphID].id = graphID;
  graphInfos[graphID].unit = unit;
  graphInfos[graphID].limits = graphLimits;

  let displayGraph;
  let enoughData = {};
  enoughData["cat1"] = false;
  enoughData["cat2"] = false;

  // d'abord on vérifie qu'on a des données pour les catégories demandées
  for (let i = 0; i < data.length; i++) {
    for (let cat of ["cat1", "cat2"]) {
      if (data[i][name[cat]]) {
        enoughData[cat] = true;
        array[cat].push(data[i][name[cat]]);
      }
    }
  }

  graphLimits["min"] = Math.min(
    Math.min(...array["cat1"]),
    Math.min(...array["cat2"])
  );
  graphLimits["max"] = Math.max(
    Math.max(...array["cat1"]),
    Math.max(...array["cat2"])
  );

  if (graphLimits["min"] > 0) {
    graphLimits["min"] = 0;
  }

  if (!enoughData["cat1"] & !enoughData["cat2"]) {
    displayGraph = false;
  } else {
    displayGraph = true;
  }

  $: scaleX = scaleTime()
    .domain(extent(data, (d) => new Date(d.year, 0)))
    .range([padding.left, width - padding.right]);

  $: scaleY = scaleLinear()
    .domain([
      graphLimits["min"],
      Math.max(0, graphLimits["max"] + graphLimits["max"] / 10),
    ])
    .range([height - padding.bottom, padding.top]);

  $: lineGenerator["cat1"] = line()
    .defined((d) => d[cat1] != null)
    .x((d) => scaleX(new Date(d.year, 0)))
    .y((d) => scaleY(d[cat1]))
    .curve(curveMonotoneX);

  $: lineGenerator["cat2"] = line()
    .defined((d) => d[cat2] != null)
    .x((d) => scaleX(new Date(d.year, 0)))
    .y((d) => scaleY(d[cat2]))
    .curve(curveMonotoneX);

  function graphMousemove(event) {
    hovered = bisect(pointer(event, this)[0]);
  }

  function graphMouseout(event) {
    hovered = null;
  }

  function bisect(mx) {
    const bisect = bisector((d) => new Date(d.year, 0)).right;
    const date = scaleX.invert(mx);
    const index = bisect(data, date, 1);
    const a = data[index - 1];
    const b = data[index];
    return b && date - new Date(a.year, 0) > new Date(b.year, 0) - date ? b : a;
  }
</script>

{#if displayGraph}
  <div class="graph-container" id="graph-{graphID}" bind:offsetWidth={width}>
    {#key $locale}
      <p class="graph-title">
        {#if enoughData["cat1"] && enoughData["cat2"]}
          <Definition name={cat1} classe="cat1" lowercase={false} />
          {$_("and")}
          <Definition name={cat2} classe="cat2" lowercase={true} />
        {:else if enoughData["cat1"]}
          <Definition name={cat1} classe="cat1" lowercase={false} />
        {:else if enoughData["cat2"]}
          <Definition name={cat2} classe="cat2" lowercase={false} />
        {/if}
      </p>
    {/key}
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <svg
      {width}
      {height}
      on:mousemove={graphMousemove}
      on:mouseout={graphMouseout}
    >
      <Axis
        {width}
        {height}
        graphInfos={graphInfos[graphID]}
        margin={padding}
        scale={scaleX}
        position="bottom"
        id="axis-x"
      />
      <Axis
        {width}
        {height}
        graphInfos={graphInfos[graphID]}
        margin={padding}
        scale={scaleY}
        position="left"
        id="axis-y"
      />
      {#each ["cat1", "cat2"] as cat}
        {#if enoughData[cat]}
          <path
            d={lineGenerator[cat](data)}
            class={cat}
            stroke-width={2.3}
            fill="none"
          />
          {#each data.filter(lineGenerator[cat].defined()) as d}
            <circle
              class={cat}
              stroke-width="2"
              stroke="white"
              r="4"
              cx={Math.min(Math.max(3, scaleX(new Date(d.year, 0))), width - 3)}
              cy={scaleY(d[name[cat]])}
            />
          {/each}
        {/if}
      {/each}
    </svg>
    {#if hovered}
      <div class="tooltip">
        {#each ["cat1", "cat2"] as cat}
          {#if hovered[name[cat]]}
            <GraphTooltip
              {cat}
              left={scaleX(new Date(hovered.year, 0))}
              top={scaleY(hovered[name[cat]])}
              top2={cat == "cat1"
                ? scaleY(hovered[name["cat2"]])
                : scaleY(hovered[name["cat1"]])}
              text={hovered[name[cat]].toLocaleString("de-DE", {
                maximumFractionDigits: 2,
              })}
              {unit}
              graphWidth={width}
            />
          {/if}
        {/each}

        {#if hovered[cat1] && hovered[cat2]}
          <div
            id="position-bar"
            style="
            top: {Math.min(scaleY(hovered[cat1]), scaleY(hovered[cat2]))}px;
            height: {height -
              padding.bottom -
              Math.min(scaleY(hovered[cat1]), scaleY(hovered[cat2]))}px;
            left: {scaleX(new Date(hovered.year, 0))}px;
            "
          />
        {:else if hovered[cat1]}
          <div
            id="position-bar"
            style="
          top: {scaleY(hovered[cat1])}px;
          height: {height - padding.bottom - scaleY(hovered[cat1])}px;
          left: {scaleX(new Date(hovered.year, 0))}px;
          "
          />
        {:else if hovered[cat2]}
          <div
            id="position-bar"
            style="
          top: {scaleY(hovered[cat2])}px;
          height: {height - padding.bottom - scaleY(hovered[cat2])}px;
          left: {scaleX(new Date(hovered.year, 0))}px;
          "
          />
        {/if}
      </div>
    {/if}
  </div>
{/if}

<style>
  .graph-container {
    position: relative;
    cursor: pointer;
    margin-top: 3em;
  }

  .graph-title {
    position: absolute;
    top: -0.5em;
    font-size: 1.2em;
  }

  :global(.cat1) {
    color: var(--primary);
  }

  :global(.cat2) {
    color: var(--secondary);
  }

  path.cat1 {
    stroke: var(--primary);
  }

  path.cat2 {
    stroke: var(--secondary);
  }

  circle.cat1 {
    fill: var(--primary);
  }

  circle.cat2 {
    fill: var(--secondary);
  }

  .tooltip {
    pointer-events: none;
  }

  #position-bar {
    position: absolute;
    width: 0;
    border-right: 1px dashed #00000050;
    z-index: -2;
  }
</style>
