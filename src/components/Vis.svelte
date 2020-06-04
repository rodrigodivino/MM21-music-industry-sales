<script>
  import { select } from "d3-selection";
  import { scaleBand, scaleLinear } from "d3-scale";
  import { extent } from "d3-array";
  import { interpolateTurbo } from "d3-scale-chromatic";

  export let sourcedata;
  const unwantedmedia = [
    "Other Tapes",
    "DVD Audio",
    "SACD",
    "Download Music Video",
    "Kiosk",
    "Other Digital"
  ];

  let problematicKey = Object.keys(sourcedata[0]).find(k => k.includes("mat"));
  for (let obj of sourcedata) obj["Format"] = obj[problematicKey];

  const data = sourcedata.filter(
    d =>
      d["Metric"] === "Value (Adjusted)" &&
      !unwantedmedia.some(m => {
        return d["Format"] === m;
      })
  );
  let width, height;
  let margin = { top: 10, left: 30, right: 10, bottom: 10 };

  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  const formats = [...new Set(data.map(d => d["Format"]))];
  const years = [...new Set(data.map(d => d["Year"]))];

  const startYears = {};
  for (let format of formats) {
    startYears[format] = {};
    const filter = data
      .filter(d => d["Format"] === format)
      .sort((a, b) => a["Year"] - b["Year"]);

    for (let entry of filter) {
      if (typeof entry["Value (Actual)"] !== "undefined") {
        startYears[format] = entry["Year"];
        break;
      }
    }
  }

  formats.sort((a, b) => startYears[a] - startYears[b]);

  $: yScale = scaleBand()
    .domain(formats)
    .range([50, innerHeight]);
  $: xScale = scaleBand()
    .domain(years)
    .range([50, innerWidth]);

  const colorScale = scaleLinear()
    .domain(extent(data.map(d => d["Value (Actual)"])))
    .range([0, 1]);

  const colorMap = v => interpolateTurbo(colorScale(v));
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

  text.y-tick {
    alignment-baseline: middle;
    text-anchor: end;
    font-family: roboto;
  }

  text.x-tick {
    font-family: monospace;
    alignment-baseline: hanging;
  }
</style>

<div bind:clientWidth={width} bind:clientHeight={height}>

  <svg>
    <g transform="translate({margin.left},{margin.top})">
      <rect width={innerWidth} height={innerHeight} fill="white" />
      <g class="y-axis">
        {#each formats as format}
          <text
            class="y-tick"
            x={xScale(startYears[format]) - 2}
            y={yScale(format)}>
            {format}
          </text>
        {/each}
      </g>
      <g class="x-axis">
        {#each years.filter(y => y % 2 == 0) as year}
          <text x={xScale(year)} y={10}>{"'" + year.toString().slice(-2)}</text>
        {/each}
      </g>

      <g class="marks">
        {#each data as entry}
          {#if entry['Value (Actual)']}
            <rect
              x={xScale(entry['Year'])}
              y={yScale(entry['Format']) - yScale.bandwidth() / 2}
              fill={colorMap(entry['Value (Actual)'])}
              width={xScale.bandwidth() + 1}
              height={yScale.bandwidth() + 1} />
          {/if}
        {/each}
      </g>
    </g>
  </svg>

</div>
