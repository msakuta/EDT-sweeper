<script>
	import { distanceFromBooleanImage } from "./distanceTransform"
	const INF = 1e20
	const width = 10
	const height = 10
	let cells = []
	let numMines = 0;
	for(let i = 0; i < height; i++){
		let row = []
		for(let j = 0; j < width; j++){
			const occupied = Math.random() < 0.07
			row.push({
				value: occupied ? 1 : 0,
				cleared: false,
				distance: occupied ? INF : 0,
			})
			if(occupied)
				numMines++;
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
	let numCleared = 0

	function clearCell(i, j){
		const neighbors = [[-1,0], [0,-1], [1,0], [0,1]];
		for(const neighbor of neighbors){
			const [i2, j2] = [i + neighbor[0], j + neighbor[1]];
			if(i2 < 0 || height <= i2 || j2 < 0 || width <= j2)
				continue;
			const cell = cells[i2][j2];
			if(!cell.cleared && 3 < cell.distance){
				cell.cleared = true;
				// Don't need assignment trigger since the caller already does that
				// cells[i2][j2] = cell;
				clearCell(i2, j2);
			}
		}
	}

	function handleMouseClick(event, i, j){
		if(gameOver)
			return;
		const cell = cells[i][j]
		if(!cell.cleared){
			cell.cleared = true;
			cells[i][j] = cell;
			clearCell(i, j);
			numCleared++;
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
			return "rgb(127,127,127)";
	}

	let cleared
	$: cleared = width * height - numCleared === numMines
</script>

<h1>EDT sweeper in Svelte</h1>

Cleared cells: {numCleared} / {width * height - numMines} Mines: {numMines}

<div style="position: relative; text-align: center; height: {height * 30}px">
	{#each cells as row, i}
		{#each row as cell, j}
			<div class="cell" style="left: {j * 30}px; top: {i * 30}px; width: 30px; height: 30px; background-color: {cellColor(cell)}"
			on:mousedown="{(event) => handleMouseClick(event, i, j)}">
				{cell.cleared ? cell.distance : ""}
			</div>
		{/each}
	{/each}
	{#if gameOver}
	<div class="message red">
		GAME OVER
	</div>
	{/if}
	{#if cleared}
	<div class="message green">
		GAME CLEAR
	</div>
	{/if}
</div>

<hr>
<div style="text-align: center">Powered by 
	<a href="https://github.com/sveltejs/svelte">Svelte</a>
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
		text-align: center;
		position: absolute;
		left: 0px;
		top: 0px;
		right: 300px;
		bottom: 300px;
	}

	.message.red{
		color: red;
	}

	.message.green{
		color: green;
	}
</style>