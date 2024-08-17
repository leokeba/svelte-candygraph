<script lang="ts">
	import Graph from '$lib/Graph.svelte';
  import type { Channel } from '$lib/types.js';
  const channels: Channel[] = [{ yVals:[], xVals:[] }];
  // Generate some x & y data.
  for (let x = 0; x <= 1; x += 0.002) {
	channels[0].xVals.push(x); // Store x-value in the channel
	channels[0].yVals.push(0.5 + 0.35 * Math.sin(x * 2 * Math.PI)); // Calculate and store y-value
  }
  // Make it scroll
  setInterval(() => {
	  let yVals = channels[0].yVals;
	  let el = yVals.shift();
	  if (el) yVals.push(el);
	  channels[0].yVals = yVals;
  }, 20);
</script>

<svelte:head>
  <title>Home</title>
  <meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
  <h1>Svelte CandyGraph Demo</h1>
  <Graph {channels}></Graph>
</section>

<style>
  section {
	  display: flex;
	  flex-direction: column;
	  justify-content: center;
	  align-items: center;
	  flex: 0.6;
  }

  h1 {
	  width: 100%;
  }
</style>