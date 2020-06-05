<script>
  import Vis from "./Vis.svelte";
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

</style>

{#if data}
  <Vis sourcedata={data} />
{:else}
  <p>Loading</p>
{/if}
