<script>
	let dices = ['⚀','⚁','⚂','⚃','⚄','⚅']
	let randomNumbers = [0,0]

	const randomize = () => {
		randomNumbers = randomNumbers.map(() => {
			return Math.floor(Math.random() * dices.length) 
		})
	}
</script>

<div class="flex w-fit mx-auto pb-8 pt-8 font-mono">
	<button class="bg-[#b0725d] text-4xl text-white px-3 py-2 hover:text-[#b0725d] hover:bg-white focus:text-[#b0725d] focus:bg-white" on:click={() => {
		randomize()
	}}>Roll</button>
	<input class="w-16 text-4xl bg-[#b0725d] text-white text-center hover:text-[#b0725d] hover:bg-white focus:text-[#b0725d] focus:bg-white" type="number" title="How many dices?" 
	value={randomNumbers.length}
	min="1"
	max="6"
		on:change={(e) => {
		if (e.target.value >= 0) {
			randomNumbers = Array.from({ length: e.target.value })
			randomize()
		}
	}}>
</div>

<div class="bg-[#b0725d] container mx-auto">
	<div class="grid grid-cols-2 mx-auto w-fit">
		{#each randomNumbers as randomNumber, index (index)}
			<div class="text-8xl text-white p-4">{dices[randomNumber]}</div>
		{/each}
	</div>
</div>