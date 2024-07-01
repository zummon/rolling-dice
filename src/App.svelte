<script>
  const dices = ["⚀", "⚁", "⚂", "⚃", "⚄", "⚅"];

  let multiple = $state(2);
  let ranNums = $derived.by(() => {
    console.log("rolling");
    return Array.from({ length: multiple }).map(() => {
      return Math.floor(Math.random() * dices.length);
    });
  });
</script>

<div class="flex w-fit mx-auto pt-8 pb-1 font-mono">
  <button
    class="text-4xl px-3 py-2 text-lime-950 bg-white"
    onclick={() => {
      multiple++;
      multiple--;
    }}>Roll</button
  >
  <input
    class="w-16 text-4xl text-center text-lime-950 bg-white"
    type="number"
    title="How many dices?"
    bind:value={multiple}
    min="1"
    max="99"
  />
</div>
<div class="text-center font-semibold pb-8">
  Score {ranNums.reduce((prev, curr) => prev + curr + 1, 0)}
</div>

<div class="flex flex-wrap gap-x-8 gap-y-4 justify-center cursor-default px-4 py-8">
  {#each ranNums as ranNum, index (index)}
    <div class="text-8xl">{dices[ranNum]}</div>
  {/each}
</div>
