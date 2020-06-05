<script>
  import { select } from "d3-selection";
  import { scaleBand, scaleLinear } from "d3-scale";
  import { extent, range } from "d3-array";
  import { interpolateTurbo as interpolation } from "d3-scale-chromatic";

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

  const colorMap = v => interpolation(colorScale(v));
</script>

<style>
  div#port {
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

  line {
    stroke: dimgray;
    stroke-width: 3;
    stroke-dasharray: 5;
  }

  text.colorLegendLabel {
    alignment-baseline: hanging;
    text-anchor: middle;
    font-family: "Courier New", Courier, monospace;
  }

  text.colorLegendTick {
    alignment-baseline: hanging;
    text-anchor: middle;
    font-family: "Courier New", Courier, monospace;
  }
</style>

<div id="port" bind:clientWidth={width} bind:clientHeight={height}>

  <svg>
    <g transform="translate({margin.left},{margin.top})">
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

      <g class="rules">
        {#each [1980, 1990, 2000, 2010] as yeartick, i}
          <line
            x1={xScale(yeartick)}
            y1={0}
            x2={xScale(yeartick)}
            y2={[yScale('Vinyl Single'), yScale('Music Video (Physical)'), yScale('Music Video (Physical)'), yScale('Ringtones & Ringbacks')][i] + yScale.bandwidth() / 2} />
        {/each}
      </g>

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

      <g class="annotations">
        <foreignObject
          x={0}
          y={innerHeight * 0.55}
          width={innerWidth * 0.5}
          height="200">
          <div
            width={innerWidth * 0.5}
            xmlns="http://www.w3.org/1999/xhtml"
            class="annotations">
            <h2>
              <b>CD: The Hero of a Generation</b>
            </h2>
            <p>
              The CD took over the industry in the early 90's,
              <br />
              reigning for
              <b>two whole decades</b>
              as Cassettes started losing space.
              <br />
              <br />
              The CD propelled the music industry to its
              <b>peak performance</b>
              in 2000.
            </p>

          </div>
        </foreignObject>
      </g>

      <g transform="translate(0,{innerHeight * 0.83})" class="colorLegend">
        {#each range(200) as i}
          <rect
            x={((innerWidth * 0.54) / 200) * i}
            width={(innerWidth * 0.54) / 200 + 1}
            height={innerHeight * 0.05}
            fill={interpolation(i / 199)} />
        {/each}
        <text
          class="colorLegendLabel"
          y={innerHeight * 0.08 + 8}
          x={(innerWidth * 0.54) / 2}>
          Revenue (Adjusted for Inflation)
        </text>
        {#each range(6) as i}
          <text
            y={innerHeight * 0.05 + 3}
            x={((innerWidth * 0.54) / 5) * i}
            class="colorLegendTick">
            ${Math.floor((i * colorScale.domain()[1]) / 5)}
          </text>
        {/each}
      </g>
    </g>
  </svg>

</div>
