<script>
  import { select, selectAll } from "d3-selection";
  import { axisBottom, axisLeft } from "d3-axis";

  import { dictionary, locale } from 'svelte-i18n';
  import { get } from 'svelte/store';

  console.log("get(dictionary)");
  console.log(get(dictionary));


  export let width;
  export let height;
  export let margin;
  export let position;
  export let scale;
  export let id;
  export let graphInfos;

  let transform;
  let g;
  
  $: {
    select(g).selectAll("*").remove();
    let axis;
    switch (position) {
      case "bottom":
        axis = axisBottom(scale).tickSizeOuter(0);
        transform = `translate(0, ${height - margin.bottom})`;
        break;
      case "left":
        axis = axisLeft(scale).tickSize(width - margin.left - margin.right);
        transform = `translate(${width - margin.right}, 0)`;
    }
    if (id == "axis-y") {
      axis.tickFormat((x, i, arr) => {
        let display;
        if (count_digits(x)) {
          display = arr[i + 1]
            ? count_digits(x)[0]
            : count_digits(x)[0] + " " + get(dictionary)[$locale][count_digits(x)[1]];
        } else {
          display = x;
        }
        return arr[i + 1] ? display : display + graphInfos.unit;
      });
      axis.ticks(5);
    }

    select(g).call(axis);

    selectAll("#graph-" + graphInfos.id + " #axis-y .tick")
      .classed("negative", (d) => d < 0)
      .filter((d) => d == 0)
      .classed("base", true)
      .classed("hide", graphInfos.limits["min"] == 0);

    selectAll("#graph-" + graphInfos.id + " #axis-y .tick text").attr(
      "x",
      (d, i, arr) => arr[i].getBBox().x + arr[i].getBBox().width * 2 + 10
    );
  }

  function count_digits(n) {
    let numDigits = 0;
    let units = {
      6: "millions",
      9: "billions",
      12: "trillions",
    };
    let integers = Math.abs(n);
    while (integers > 0) {
      integers = (integers - (integers % 10)) / 10;
      numDigits++;
    }
    let u = Math.floor(numDigits / 3) * 3;
    let r = numDigits % 3;
    let number = n;
    let unit = "";
    if (u > 3) {
      number = n / Number("1.0e+" + u);
      unit = units[u];
      if (number == 1 || number == 0) {
        unit = unit.substring(0, unit.length - 1);
      }
      return [number, unit];
    } else {
      return false;
    }
  }
</script>

<g class="axis" bind:this={g} {transform} {id} />

<style global>
  :global(#axis-y .domain) {
    display: none;
  }

  :global(#axis-x .domain) {
    stroke-width: 1.4;
  }

  :global(#axis-y .tick:not(.base) line) {
    stroke: #00000050;
    stroke-dasharray: 2, 2;
  }

  :global(#axis-y .tick.base line) {
    stroke: #00000050;
    stroke-dasharray: 6, 2;
  }

  :global(.tick text) {
    font-size: 16px;
    font-family: var(--font-family-sans-serif);
  }

  :global(#axis-x .tick text) {
    font-weight: 500;
  }

  :global(#axis-x .tick:first-of-type text) {
    transform: translateX(1em);
  }

  :global(#axis-x .tick:last-of-type text) {
    transform: translateX(-1em);
  }

  :global(#axis-y .tick text) {
    fill: #00000050;
    paint-order: stroke;
    stroke: #fff;
    stroke-width: 3px;
  }

  :global(#axis-y .tick.negative text) {
    fill: var(--primary);
  }

  /* :global(#axis-y .tick.base text) {
    fill: #000;
    transform: translateY(-0.8em);
  } */

  :global(.tick.base line) {
    stroke-dasharray: 0;
    stroke-width: 1.33;
    stroke: #000;
  }

  :global(#axis-y .tick.hide text) {
    display: none;
  }

  :global(#axis-x .tick line) {
    stroke-width: 1.33;
    stroke: #000;
  }

</style>
