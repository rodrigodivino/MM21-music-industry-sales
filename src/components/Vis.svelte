<script>
  import { select } from "d3-selection";
  import { scaleBand, scaleLinear } from "d3-scale";
  import { extent, range } from "d3-array";
  import { interpolateTurbo as interpolation } from "d3-scale-chromatic";
  import { annotation } from "d3-svg-annotation";

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

  $: colorScale = scaleLinear()
    .domain(extent(data.map(d => d["Value (Actual)"])))
    .range([0, 1]);

  $: colorlegendwidth = innerWidth * 0.54;
  $: colorLegendScale = scaleLinear()
    .domain(extent(data.map(d => d["Value (Actual)"])))
    .range([0, colorlegendwidth]);

  const colorMap = v => interpolation(colorScale(v));

  let annotationsGroup;
  $: annotations = [
    {
      note: {
        label: `The CD took over the industry in the early 90's,
          reigning for two whole decades as Cassettes started losing space.`,
        title: "The Hero of a Generation",
        align: "middle",
        wrap: innerWidth * 0.4
      },
      connector: {
        end: "dot", // none, or arrow or dot
        type: "line", // Line or curve
        points: 1, // Number of break in the curve
        lineType: "vertical"
      },
      x: xScale(startYears["CD"]) - 30,
      y: yScale("CD"),
      dy: 0.15 * innerWidth,
      dx: -0.05 * innerWidth
    }
  ];
  $: makeAnnotations = annotation().annotations(annotations);
  $: if ((makeAnnotations, annotationsGroup))
    select(annotationsGroup).call(makeAnnotations);
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
    font-family: sans-serif;
    font-size: 0.8em;
  }

  path.arrow {
    stroke: black;
    stroke-width: 1;
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

      <g class="annotations" bind:this={annotationsGroup} />
      <!-- <g class="story">
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
      </g> -->

      <g transform="translate(0,{innerHeight * 0.83})" class="colorLegend">
        {#each range(200) as i}
          <rect
            x={(colorlegendwidth / 200) * i}
            width={colorlegendwidth / 200 + 1}
            height={innerHeight * 0.05}
            fill={interpolation(i / 199)} />
        {/each}
        <text
          class="colorLegendLabel"
          y={innerHeight * 0.08 + 8}
          x={(innerWidth * 0.54) / 2}>
          Revenue (Adjusted for Inflation)
        </text>
        {#each range(11) as i}
          <text
            y={innerHeight * 0.05 + 3}
            x={colorLegendScale(i * 2000)}
            class="colorLegendTick">
            ${i * 2}M
          </text>
        {/each}
      </g>
    </g>
  </svg>

</div>
