<script>
	let cells = [[]]
	for(let i = 0; i < 10; i++){
		let row = []
		for(let j = 0; j < 10; j++){
			row.push(Math.random() < 0.02 ? "1" : "0")
		}
		cells.push(row)
	}
</script>

<h1>EDT sweeper in Svelte</h1>

<div style="position: relative">
	{#each cells as row, i}
		{#each row as cell, j}
			<div class="cell" style="left: {j * 30}px; top: {i * 30}px; width: 30px; height: 30px; ">
				{cell}
			</div>
		{/each}
	{/each}
</div>

<style>
	.cell{
		text-align: center;
		position: absolute;
		border: solid 1px black;
	}
</style>