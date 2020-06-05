<script>
  import Vis from "../components/Vis.svelte";
  import { read as readXLSX, utils } from "xlsx";
  import { onMount } from "svelte";

  let data;
  onMount(async () => {
    const res = await fetch(
      "https://query.data.world/s/ne5hioeoptdqvimde7lx4l262nr7i3"
    );
    const buffer = await res.arrayBuffer();
    const workbook = readXLSX(new Uint8Array(buffer), {
      type: "array"
    });
    data = utils.sheet_to_json(workbook.Sheets.Sheet1);
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
<h2>The rise and fall of music media formats</h2>

{#if data}
  <Vis sourcedata={data} />
{:else}
  <p>Loading</p>
{/if}
