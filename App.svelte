<script>
	import * as d3 from "d3";
	import AxisHorizontal from "./AxisHorizontal.svelte";
	import AxisVertical from "./AxisVertical.svelte";

	// access data
	let data = [];
	d3.csv(
	  "https://raw.githubusercontent.com/mweers/mweers.github.io/master/weight.csv"
	).then(res => {
	  data = res.slice();

	  // data = res.filter(obj => {
	  // 	return obj.date > new Date ("1/1/2018")
	  // });
	  //console.log(new Date(data[0].date));
	});

	data = data.filter(obj => {
	  return obj.date > new Date("1/1/2015");
	});

	const xAccessor = d => new Date(d.date);
	const yAccessor = d => parseInt(d.weight);

	// create chart dimensions
	let margin = {
	  left: 30,
	  top: 0,
	  right: 0,
	  bottom: 30
	};
	let width = 400;
	let height = 100;
	$: boundsWidth = width - margin.left - margin.right;
	$: boundsHeight = height - margin.top - margin.bottom;

	// create tooltip

	$: tooltip = d3
	  .select("body")
	  .append("div")
	  .style("position", "absolute")
	  .style("visibility", "hidden");

	// create scales
	$: xScale = d3
	  .scaleTime()
	  .domain(d3.extent(data, xAccessor))
	  .range([0, boundsWidth]);

	$: yScale = d3
	  .scaleLinear()
	  .domain(d3.extent(data, yAccessor))
	  .range([boundsHeight, 0])
	  .nice(true);

	// draw data
	$: lineD = d3
	  .line()
	  .x(d => xScale(xAccessor(d)))
	  .y(d => yScale(yAccessor(d)))
	  .curve(d3.curveNatural)(data);
</script>

<figure>
<div class="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
	<svg width={width} height={height}>
		<g transform="translate({margin.left}, {margin.top})">
			<path
				d={lineD}
				fill="none"
				stroke="palevioletred"
				stroke-width="3"
			/>

			{#each data as d}
				<circle
					cx={xScale(xAccessor(d))}
					cy={yScale(yAccessor(d))}
					r="4"
					fill="#292421"
				/>
			{/each}
		</g>
		<g transform="translate({margin.left}, {boundsHeight + margin.top})">
			<AxisHorizontal
				scale={xScale}
				count="5"
				format={d3.timeFormat("%-m/%d/%y")}
			/>
		</g>
		<g transform="translate({margin.left}, 0)">
			<AxisVertical
				count="5"
				scale={yScale}
			/>
		</g>
	</svg>
</div>
</figure>

<style>
	.wrapper {
	  margin: 0;
	  font-family: sans-serif;
	  width: 100%;
	  height: 300px;
	  min-width: 0;
	}

	svg {
	  overflow: visible;
	}
</style>