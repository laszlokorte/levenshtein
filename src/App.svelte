<script>
	let wordA = 'hello';
	let wordB = 'world';
	
	$: charsA = ['', ...wordA.split('')]
	$: charsB = ['', ...wordB.split('')]
	
	$: lengthA = charsA.length
	$: lengthB = charsB.length
	
	$: dpMatrix = Array(lengthB).fill(null).map((_,row) => Array(lengthA).fill(null).map((_,col) => row===0?col:col==0?row:null))
	
	$: opMatrix = Array(lengthB).fill(null).map((_,row) => Array(lengthA).fill(null).map((_,col) => row==col?null:row===0?'delete':col==0?'insert':null))
	
	$: for(let r=1;r<lengthB;r++) {
		 for(let c=1;c<lengthA;c++) {
			 const insertCost = dpMatrix[r-1][c]
			 const replaceCost = dpMatrix[r-1][c-1]
			 const deleteCost = dpMatrix[r][c-1]
			 const minCost = Math.min(insertCost, replaceCost, deleteCost)
			 const isSame = charsA[c]==charsB[r]
			 
			 
			 dpMatrix[r][c] = isSame ? replaceCost : minCost + 1
			 
			 if(insertCost === minCost) {
				 opMatrix[r][c] = 'insert'
			 } else if(replaceCost === minCost) {
				 opMatrix[r][c] = 'replace'
			 } else {
				 opMatrix[r][c] = 'delete'
			 }
		 }
	}
	
	$: opsArray = Array(lengthA+lengthB).fill(null).reduce(({r,c,ops}) => {
		if(c==0 && r==0) {
			return {r,c,ops}
		}
		const isSame = charsA[c]==charsB[r]
		if(opMatrix[r][c]==='insert') {
			return {r:r-1,c:c,ops:[...ops,{op:'insert',char:charsB[r]}]}
		} else if(opMatrix[r][c]==='replace') {
			return {r:r-1,c:c-1,ops:isSame?[...ops,{op:'keep', char:charsA[c]}]:[...ops,{op:'replace',old:charsA[c],new:charsB[r]}]}
		} else if(opMatrix[r][c]==='delete') {
			return {r:r,c:c-1,ops:[...ops,{op:'delete',char:charsA[c]}]}
		} else {
			return {r,c,ops}
		}
	}, {r: lengthB-1, c:lengthA-1, ops: []}).ops.reverse()
	
	$: mergedOps = opsArray.reduce((acc, op) => {
		if(acc.length) {
			const prevOp = acc[acc.length-1]
			if(prevOp.op == op.op) {
				const mergedOp = {
					op: op.op,
				}
				
				if(op.char) {
					mergedOp.char = prevOp.char + op.char
				} else {
					mergedOp.old = prevOp.old + op.old
					mergedOp.new = prevOp.new + op.new
				}
				
				return [
					...acc.slice(0,-1),
					mergedOp
				]
			} else {
				return [...acc, op]
			}
		} else {
			return [op]
		}
	}, [])
	
	$: distance = dpMatrix[lengthB-1][lengthA-1]
</script>
<style>
	h1 {
		margin: 0
	}
	
	article {
		max-width: 100ch;
		margin: auto;
	}
	
	th, td {
		width: 3ch;
		height:3ch;
		text-align: center;
		vertical-align: middle;
	}
	
	.op-insert {
		position: relative;
		color: green;
		background: #efe;
	}
	
	.op-delete {
		position: relative;
		color: red;
		background: #fee;
	}
	
	.op-replace {
		position: relative;
		color: blue;
		background: #eef;
	}
	
	.op-insert::before {
		content: '';
		border-width: 5px;
		border-style: solid;
		display: block;
		width:0;
		height:0;
		border-color: transparent transparent currentColor transparent;
		margin: auto;
		position: absolute;
		top: 0;
		left: 50%;
		margin-left: -5px;
		border-top-width: 0;
	}
	
	.op-delete::before {
		content: '';
		border-width: 5px;
		border-left-width: 0;
		border-style: solid;
		display: block;
		width:0;
		height:0;
		border-color: transparent currentColor transparent transparent;
		margin: auto;
		position: absolute;
		left: 0;
		top: 50%;
		margin-top: -5px;
		vertical-align: middle;
	}
	
	.op-replace::before {
		content: '';
		border-width: 4px;
		border-style: solid;
		display: block;
		width:0;
		height:0;
		border-color: currentColor transparent transparent currentColor;
		margin: auto;
		position: absolute;
		left: 2px;
		top: 2px;
	}
	
	del {
		color: darkred;
		background: #fcc;
	}
	
	ins {
		color: darkgreen;
		background: #cfc;
	}
	
	dl {
		display: grid;
		grid-template-columns: auto auto;
		justify-content: start;
		margin: 0;
		padding: 0;
		gap: 0.2em 1em;
		align-items: baseline;
	}
	
	dd {
		margin: 0;
	}

	.noincr {
		counter-increment: none;
	}
	.noincr::before {
		content: "(...";
	}
	.noincr::after {
		content: ")";
	}

	.incr {
		counter-increment: op;
	}

	.incr::before {
		content: counter(op) ". ";
	}

	.op-list {
		list-style: none;
		counter-reset: op;
		padding: 0;
	}
</style>
<article>

<h1>
	Levenshtein Distance
</h1>

<p>
	The Levenshtein Distance or edit distance measures the similarity between two sequences/strings. The distance is measured in number of single character operations (delete, insert, replace) needed to transform one sequence into another.
</p>

<fieldset>
	<legend>
		Input
	</legend>
	<dl>
		<dt><label for="word-a">Word A</label></dt>
		<dd><input id="word-a" type="text" maxlength="20" length="10" bind:value={wordA} /></dd>
		<dt><label for="word-b">Word B</label></dt>
		<dd><input id="word-b" type="text" maxlength="20" length="10" bind:value={wordB} /></dd>
		<dt>Levenstein Distance</dt>
		<dd>{distance}</dd>
	</dl>
</fieldset>

<fieldset>
	<legend>
		Dynamic Programming Steps
	</legend>
	
	<table>
		<thead>
			<tr>
				<th></th>
				{#each charsA as char}
				<th>&laquo;{char}&raquo;</th>
				{/each}
			</tr>
		</thead>

		<tbody>
				{#each charsB as char, r}
			<tr>
				<th>&laquo;{char}&raquo;</th>
				{#each dpMatrix[r] as result, c}
				<td class={'op-'+opMatrix[r][c]}>{result}</td>
				{/each}
			</tr>
				{/each}
		</tbody>
	</table>
</fieldset>

<fieldset>
	<legend>
		Operations for changing word A into word B
	</legend>

	<ol class="op-list">
		{#each opsArray as op}
	{#if op.op == 'replace'}
	<li class="incr">
	{op.op}: &laquo;<strong>{op.old}</strong>&raquo; by &laquo;<strong>{op.new}</strong>&raquo;
	</li>
	{:else if op.op=='keep'}
	<li class="noincr" style="list-style: none;">
	{op.op}: &laquo;<strong>{op.char}</strong>&raquo;
	</li>
	{:else}
	<li class="incr">
	{op.op}: &laquo;<strong>{op.char}</strong>&raquo;
	</li>
	{/if}
	{/each}
	</ol>
	
<h3>Diff</h3>
		
<pre>
{#each mergedOps as op}
{#if op.op == 'keep'}
{op.char}
{:else if op.op == 'replace'}
<del>{op.old}</del><ins>{op.new}</ins>
{:else if op.op == 'delete'}
<del>{op.char}</del>
{:else if op.op == 'insert'}
<ins>{op.char}</ins>
{/if}
{/each}
</pre>
	
</fieldset>

</article>