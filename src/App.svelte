<script>
	import { onMount } from "svelte"

	let dices = $state(["⚀", "⚁", "⚂", "⚃", "⚄", "⚅"])
	let score = $state(0)
	let multiple = $state(2)
	let rolled = $state([])

	function drop () {
		score = 0
		rolled = []
		Array.from({ length: multiple }).map(() => {
			let index = Math.floor(Math.random() * dices.length)
			score += index + 1
			rolled.push({
				index: index,
				rotate: Math.floor(Math.random() * 180)
			})
		})
	}

	onMount(() => {
		let params = new URLSearchParams(location.search)
    let count = params.get('count')
		if (count && !isNaN(count) && +count <= 99 && +count >= 1) {
			multiple = +count
		}
		drop()
	})
</script>

<div class="flex flex-wrap gap-4 items-center ">
	<input class="w-16 text-center font-semibold text-2xl border-fuchsia-500 border-b-2 bg-transparent" type="number" title="How many dices?" min="1" max="99" bind:value={multiple}>
	<button class="px-3 font-semibold text-2xl border-fuchsia-500 border-b-2 cursor-pointer" title="Click to play" onclick={()=> { drop() }}>
		Roll
	</button>
	<div class="text-center text-2xl font-mono">
		Score {score}
	</div>
</div>

<div class="grow flex flex-wrap gap-x-8 gap-y-4 justify-center items-center cursor-default px-4 py-8 overflow-hidden">
	{#each rolled as placed, idx (idx)}
		<div class="text-9xl" style="transform: rotate({placed.rotate}deg)">{dices[placed.index]}</div>
	{/each}
</div>