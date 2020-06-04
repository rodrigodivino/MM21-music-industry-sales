<script>
  import Vis from "../components/Vis.svelte";
  import { read as readXLSX, utils } from "xlsx";
  let dataPromise = fetch(
    "https://query.data.world/s/ne5hioeoptdqvimde7lx4l262nr7i3"
  )
    .then(res => {
      return res.arrayBuffer();
    })
    .then(res => {
      var workbook = readXLSX(new Uint8Array(res), {
        type: "array"
      });
      return utils.sheet_to_json(workbook.Sheets.Sheet1);
    });
</script>

<style>
  h1 {
    font-size: 2.8em;
    text-transform: uppercase;
    font-weight: 700;
    margin: 0 2em 0.5em 2em;
    text-align: center;
  }

  h2 {
    font-size: 2em;
    text-transform: uppercase;
    font-weight: 700;
    margin: 0 2em 0.5em 2em;
    color: dimgray;
    text-align: center;
  }
</style>

<svelte:head>
  <title>Music Industry</title>
</svelte:head>

<h1>A Battle of Media!</h1>
<h2>A history of music media formats and how they impacted industry</h2>

{#await dataPromise}
  <p>Loading</p>
{:then data}
  <Vis {data} />
{:catch}
  <p>Error, unable to fetch data</p>
{/await}
