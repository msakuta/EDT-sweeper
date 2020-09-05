<script>
	import { distanceFromBooleanImage } from "./distanceTransform"
	const INF = 1e20
	const width = 10
	const height = 10
	let cells = []
	for(let i = 0; i < height; i++){
		let row = []
		for(let j = 0; j < width; j++){
			const occupied = Math.random() < 0.07
			row.push({
				value: occupied ? 1 : 0,
				cleared: false,
				distance: occupied ? INF : 0,
			})
		}
		cells.push(row)
	}

	let booleanImage = [];
	for(let i = 0; i < height; i++){
		for(let j = 0; j < width; j++){
			booleanImage.push(cells[i][j].value);
		}
	}
	let dt = distanceFromBooleanImage(booleanImage, width, height, "EDT");
	for(let i = 0; i < height; i++){
		for(let j = 0; j < width; j++){
			cells[i][j].distance = dt[i * width + j].toFixed(2);
		}
	}

	let gameOver = false
	let cleared = 0

	function handleMouseClick(event, i, j){
		if(gameOver)
			return;
		const cell = cells[i][j]
		if(!cell.cleared){
			cell.cleared = true;
			cells[i][j] = cell
			cleared++;
			if(cell.value){
				gameOver = true;
			}
		}
	}

	let maxDist
	$: maxDist = cells.reduce((accum, row) => row.reduce(
		(accum, cell) => Math.max(accum, cell.distance), accum), 0)

	function cellColor(cell){
		if(cell.cleared){
			const value = cell.distance / maxDist * 191 + 63;
			return `rgb(255, ${value}, ${value})`;
		}
		else
			return "rgb(127,127,127";
	}
</script>

<h1>EDT sweeper in Svelte</h1>

Cleared cells: {cleared}

<div style="position: relative">
	{#each cells as row, i}
		{#each row as cell, j}
			<div class="cell" style="left: {j * 30}px; top: {i * 30}px; width: 30px; height: 30px; background-color: {cellColor(cell)}"
			on:mousedown="{(event) => handleMouseClick(event, i, j)}">
				{cell.cleared ? cell.distance : ""}
			</div>
		{/each}
	{/each}
	{#if gameOver}
	<div class="message">
		GAME OVER
	</div>
	{/if}
</div>

<style>
	.cell{
		text-align: center;
		position: absolute;
		border: solid 1px black;
		font-size: 7pt;
	}

	.message{
		font-size: 30pt;
		color: red;
		text-align: center;
		position: absolute;
		left: 0px;
		top: 0px;
		right: 300px;
		bottom: 300px;
	}
</style>