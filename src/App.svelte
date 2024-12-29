<script>
	import { onMount } from "svelte"

	let dices = $state(["⚀", "⚁", "⚂", "⚃", "⚄", "⚅"])
	let multiple = $state(2)

	let rolled = $derived.by(() => {
		let score = 0
		let prepare = Array.from({ length: multiple }).map(() => {
			let index = Math.floor(Math.random() * dices.length)
			score += index + 1
			return {
				index: index,
				rotate: Math.floor(Math.random() * 180),
			}
		})
		return {
			score: score,
			dices: prepare,
		}
	})

	function drop () {
		multiple += 1
		multiple += -1
	}

	onMount(() => {
		
	})
</script>

<div class="flex flex-wrap gap-4 items-center ">
	<button class="px-3 font-semibold text-2xl border-fuchsia-500 border-b-2 cursor-pointer" title="Click to play" onclick={()=> { drop() }}>
		Roll
	</button>
	<input class="w-16 text-center font-semibold text-2xl border-fuchsia-500 border-b-2 bg-transparent" type="number" title="How many dices?" min="1" max="99" bind:value={multiple}>
	<div class="text-center text-2xl font-mono">
		Score {rolled.score}
	</div>
</div>

<div class="grow flex flex-wrap gap-x-8 gap-y-4 justify-center items-center cursor-default px-4 py-8 overflow-hidden">
	{#each rolled.dices as dice}
		<div class="text-9xl" style="transform: rotate({dice.rotate}deg)">{dices[dice.index]}</div>
	{/each}
</div>