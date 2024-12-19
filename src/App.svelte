<script>
	import { onMount } from "svelte";

	let dices = $state(["⚀", "⚁", "⚂", "⚃", "⚄", "⚅"]);

	let multiple = $state(2)
	let ranNums = $state([])

	function ranNumsFunc() {
		console.log("rolling");
		ranNums = Array.from({ length: multiple }).map(() => {
			return {
				value: Math.floor(Math.random() * dices.length),
				rotate: Math.floor(Math.random() * 180)
			};
		});
	}

	onMount(() => {
		ranNumsFunc();
	})
</script>

<div class="flex w-fit mx-auto pt-8 pb-1 font-mono">
	<button class="text-4xl px-3 py-2 text-lime-950 bg-white" onclick={()=> { ranNumsFunc() }}>
		Roll
	</button>
	<input class="w-16 text-4xl text-center text-lime-950 bg-white" type="number" title="How many dices?" min="1"
		max="99" value={multiple} oninput={(e)=> { multiple = Number(e.target.value) ; ranNumsFunc() }} >
</div>
<div class="text-center font-semibold pb-8">
	Score {ranNums.reduce((prev, curr) => prev + curr.value + 1, 0)}
</div>

<div class="flex flex-wrap gap-x-8 gap-y-4 justify-center cursor-default px-4 py-8">
	{#each ranNums as ranNum}

	<div class="text-9xl" style={`transform: rotate(${ranNum.rotate}deg)`}>{dices[ranNum.value]}</div>
	{/each}
</div>