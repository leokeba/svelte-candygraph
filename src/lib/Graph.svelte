<script lang="ts">
	import { onMount } from 'svelte';
	import type { Channel } from '$lib/types.d.ts'; 
	import { 
		CandyGraph,
		createDefaultFont,
		LinearScale,
		LineStrip,
		OrthoAxis,
		CartesianCoordinateSystem,
		type OrthoAxisOptions,
		type Viewport,
		CoordinateSystem,
	} from "candygraph";
	
	let cg: CandyGraph;
	let canvas: HTMLCanvasElement;
	export let xAxis: OrthoAxisOptions = {
		labelSide: 1,
		labelSize: 24,
		tickOffset: -2.5,
		tickLength: 8,
		tickStep: 0.1,
		tickWidth: 2,
		axisWidth: 2,
		labelFormatter: (n) => n.toFixed(1),
	};
	export let yAxis: OrthoAxisOptions = {
		labelSize: 24,
		tickOffset: 2.5,
		tickLength: 8,
		tickStep: 0.1,
		tickWidth: 2,
		axisWidth: 2,
		labelFormatter: (n) => n.toFixed(1),
	};
	export let channels: Channel[] = [{ xVals: [], yVals: [] }]; // Initialize the channel
	$: if (cg) {
		requestAnimationFrame(render);
		channels;
	}
	
	let lastRender = 0;
	export let maxRate = 16.66;
	
	onMount(() => {
		cg = new CandyGraph({canvas: canvas});
		// cg = new CandyGraph();
		const resizeObserver = new ResizeObserver(entries => {
			const entry = entries.at(0);
			if (entry) {
				canvas.width = entry.contentRect.width * devicePixelRatio;
				canvas.height = entry.contentRect.height * devicePixelRatio;
				requestAnimationFrame(render);
			}
		});
		resizeObserver.observe(canvas);
	});
	
	async function render(time: number = 0) {
		if (time - lastRender < maxRate) return; // skip render if above max rate
		lastRender = time;
		const viewport: Viewport = {
			x: 0,
			y: 0,
			width: cg.canvas.width,
			height: cg.canvas.height,
		};
		const coords = new CartesianCoordinateSystem(
		cg,
		new LinearScale([0, 1], [48, viewport.width - 24]),
		new LinearScale([0, 1], [40, viewport.height - 24])
		);
		
		const font = await createDefaultFont(cg);
		cg.clear([1, 1, 1, 1]);
		
		let renderables = [
		...channels.map((channel, index) => {
			return new LineStrip(cg, channel.xVals, channel.yVals, {
				colors: [1, 0.5, 0.0, 1.0],
				widths: 5,
			})}),
			new OrthoAxis(cg, coords, "x", font, xAxis),
			new OrthoAxis(cg, coords, "y", font, yAxis)
			];
			cg.render(coords, viewport, renderables);
			renderables.forEach((renderable) => {
				renderable.dispose();
			});
			// cg.copyTo(viewport, canvas);
	}
		
</script>
	
<div id="candygraph-container">
	<canvas bind:this={canvas} id="candygraph-canvas"></canvas>
</div>

<style>
	#candygraph-container {
		width: 100%;
		height: 100%;
	}
	#candygraph-container canvas {
		width: 100%;
		height: 100%;
		background: #FFF;
	}
</style>