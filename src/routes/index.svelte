<script>
  import Fetcher from "../components/Fetcher.svelte";
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

<Fetcher />
