<script>
	const INF = 1e20
	const width = 10
	const height = 10
	let cells = []
	for(let i = 0; i < height; i++){
		let row = []
		for(let j = 0; j < width; j++){
			const occupied = Math.random() < 0.02
			row.push({
				value: occupied ? "1" : "0",
				cleared: false,
				distance: occupied ? INF : 0,
			})
		}
		cells.push(row)
	}


	// https://observablehq.com/@mourner/fast-distance-transform
	const v = new Array(width, 0);
	const z = new Array(width, 0);
	const f = new Array(width, 0);

	const edt1d = (gridReader, gridWriter, offset, length) => {
		v[0] = 0;
		z[0] = -INF;
		z[1] = INF;
		for (let q = 0; q < length; q++) f[q] = gridReader(offset);

		for (let q = 1, k = 0, s = 0; q < length; q++) {
			do {
			const r = v[k];
			s = (f[q] - f[r] + q * q - r * r) / (q - r) / 2;
			} while (s <= z[k--]);

			k += 2;
			v[k] = q;
			z[k] = s;
			z[k + 1] = INF;
		}

		for (let q = 0, k = 0; q < length; q++) {
			while (z[k + 1] < q) k++;
			const r = v[k];
			gridWriter(offset, f[r] + (q - r) * (q - r));
		}
	}

	(() => {
		for (let x = 0; x < width-1; x++) { // transform each column
			edt1d((y) => cells[y][x].distance, (y, v) => cells[y][x].distance = v, x, height-1);
		}
		for (let y = 0; y < height-1; y++) { // transform each row
			edt1d((x) => cells[y][x].distance, (x, v) => cells[y][x].distance = v, y, width-1);
		}
	})()

	let cleared = 0

	function handleMouseClick(event, i, j){
		const cell = cells[i][j]
		if(!cell.cleared){
			cell.cleared = true;
			cells[i][j] = cell
			cleared++;
		}
	}
</script>

<h1>EDT sweeper in Svelte</h1>

Cleared cells: {cleared}

<div style="position: relative">
	{#each cells as row, i}
		{#each row as cell, j}
			<div class="cell" style="left: {j * 30}px; top: {i * 30}px; width: 30px; height: 30px; "
			on:mousedown="{(event) => handleMouseClick(event, i, j)}">
				{cell.cleared ? cell.distance : ""}
			</div>
		{/each}
	{/each}
</div>

<style>
	.cell{
		text-align: center;
		position: absolute;
		border: solid 1px black;
		font-size: 7pt;
	}
</style>