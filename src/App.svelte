<script>
	import {onMount} from 'svelte'
	let dices = ['⚀','⚁','⚂','⚃','⚄','⚅']
	let randomNumbers = [0]

	const randomize = () => {
		randomNumbers = randomNumbers.map(() => {
			return Math.floor(Math.random() * dices.length) 
		})
	}
	onMount(() =>{
		randomize()
	})
</script>

<div class="flex w-fit mx-auto mb-8 font-mono">
	<button class="text-4xl px-3 py-2 text-[#b0725d] bg-white " on:click={() => {
		randomize()
	}}>Roll</button>
	<input class="w-16 text-4xl text-center text-[#b0725d] bg-white " type="number" title="How many dices?" 
	value={randomNumbers.length}
	min="1"
	max="8"
		on:change={(e) => {
		if (e.target.value >= 0) {
			randomNumbers = Array.from({ length: e.target.value })
			randomize()
		}
	}}>
</div>

<div class="bg-[#b0725d] container mx-auto">
	<div class="grid grid-cols-2 mx-auto w-fit cursor-default">
		{#each randomNumbers as randomNumber, index (index)}
			<div class="text-8xl text-white p-4">{dices[randomNumber]}</div>
		{/each}
	</div>
</div>

<div class="text-center mt-8 text-white">
	Made by zummon (Teerapat Anantarattanachai)<br>
	Something breaks, shows incorrect result. Let me know.
</div>