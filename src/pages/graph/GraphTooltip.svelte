<script>
  import { afterUpdate } from "svelte";

  export let cat;
  export let left;
  export let top;
  export let top2;
  export let text;
  export let unit;
  export let graphWidth;

  let tooltipText;
  let translateX;
  let translateY;

  afterUpdate(() => {
    let posLeft = tooltipText.style.left;

    posLeft = Number(posLeft.substring(0, posLeft.length - 2));

    translateX = 0;
    translateY = 0;

    if (graphWidth - (posLeft + tooltipText.offsetWidth) < 0) {
      translateX = -tooltipText.offsetWidth - 8;
    }

    if (Math.abs(top - top2) < 20) {
      if (top > top2) {
        translateY = 20 - (top - top2);
      }
    }

    tooltipText.style.transform = `translate(${translateX}px, ${translateY}px)`;
  });
</script>

<div class="tooltip-{cat}">
  <div
    bind:this={tooltipText}
    class="tooltip-text {cat}"
    style="left: {left + 4}px; top: {top}px;"
  >
    <p>
      {text}{unit}
    </p>
  </div>
</div>

<style>
  .tooltip-point {
    position: absolute;
    height: 10px;
    width: 10px;
    background-color: var(--primary);
    border-radius: 50%;
    border: 2px solid white;
  }
  
  .tooltip-point.cat2 {
    background-color: var(--secondary);
  }

  .tooltip-text {
    position: absolute;
    font-weight: 500;
    text-shadow: 2px 0 0 #fff, -2px 0 0 #fff, 0 2px 0 #fff, 0 -2px 0 #fff,
      1px 1px #fff, -1px -1px 0 #fff, 1px -1px 0 #fff, -1px 1px 0 #fff;
  }
</style>
