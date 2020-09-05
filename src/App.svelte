<script>
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

	// Code borrowed from: https://github.com/parmanoir/Meijster-distance
	// Meijster distance
	//	method : 'EDT' (Euclidean) 'MDT' (Manhattan) 'CDT' (Chessboard)
	//	Everything is integer math, hence Math.floor()
	function distanceFromBooleanImage(booleanImage, m, n, method) {
		// First phase
		var infinity = m+n
		var b = booleanImage
		var g = new Array(m*n)
		for (var x=0; x<m; x++)	{
			if (b[x+0*m])
				g[x+0*m] = 0
			else
				g[x+0*m] = infinity
			// Scan 1
			for (var y=1; y<n; y++)	{
				if (b[x+y*m])
					g[x+y*m] = 0
				else
					g[x+y*m] = 1 + g[x+(y-1)*m]
			}
			// Scan 2
			for (var y=n-1; y>= 0; y--)	{
				if (g[x+(y+1)*m] < g[x+y*m])
					g[x+y*m] = 1 + g[x+(y+1)*m]
			}
		}

		// Euclidean
		function EDT_f(x, i, g_i) {	
			return (x-i)*(x-i) + g_i*g_i	
		}
		function EDT_Sep(i, u, g_i, g_u) {	
			return Math.floor((u*u - i*i + g_u*g_u - g_i*g_i)/(2*(u-i)))	
		}

		// Manhattan
		function MDT_f(x, i, g_i) {	
			return Math.abs(x-i) + g_i		
		}
		function MDT_Sep(i, u, g_i, g_u) {
			if (g_u >= (g_i + u - i))
				return infinity
			if (g_i > (g_u + u - i))
				return -infinity
			return Math.floor((g_u - g_i + u + i)/2)
		}

		// Chessboard
		function CDT_f(x, i, g_i) {	
			return Math.max(Math.abs(x-i), g_i)	
		}
		function CDT_Sep(i, u, g_i, g_u) {
			if (g_i <= g_u)
				return Math.max(i+g_u, Math.floor((i+u)/2))
			else
				return Math.min(u-g_i, Math.floor((i+u)/2))
		}

		// Second phase
		var f	= EDT_f
		var Sep	= EDT_Sep
		var dt	= new Array(m*n)
		var s	= new Array(m)
		var t	= new Array(m)
		var q	= 0
		var w
		for (var y=0; y<n; y++)	{
			q = 0
			s[0] = 0
			t[0] = 0
			
			// Scan 3
			for (var u=1; u<m; u++)	{
				while (q >= 0 && f(t[q], s[q], g[s[q]+y*m]) > f(t[q], u, g[u+y*m]))
					q--
				if (q < 0)	{
					q = 0
					s[0] = u
				}	else	{
					w = 1 + Sep(s[q], u, g[s[q]+y*m], g[u+y*m])
					if (w < m)	{
						q++
						s[q] = u
						t[q] = w
					}
				}
			}
			// Scan 4
			for (u=m-1; u>=0; u--)	{
				var d = f(u, s[q], g[s[q]+y*m])
				if (method == 'EDT')		
					d = /*Math.floor*/(Math.sqrt(d))
				dt[u+y*m] = d
				if (u == t[q])
					q--
			}
		}
		return dt
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