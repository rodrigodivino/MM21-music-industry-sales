<script>
  import { select } from "d3-selection";
  import { scaleBand } from "d3-scale";

  export let data;

  let width, height;
  let margin = { top: 10, left: 10, right: 10, bottom: 10 };

  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  let problematicKey = Object.keys(data[0]).find(k => k.includes("mat"));
  for (let obj of data) obj["Format"] = obj[problematicKey];
</script>

<style>
  div {
    width: 90vw;
    height: 80vh;
    margin: auto;
  }

  svg {
    width: 100%;
    height: 100%;
  }
</style>

<div bind:clientWidth={width} bind:clientHeight={height}>

  <svg>
    <rect {width} {height} />
    <g transform="translate({margin.left},{margin.top})">
      <rect width={innerWidth} height={innerHeight} fill="white" />
    </g>
  </svg>

</div>
