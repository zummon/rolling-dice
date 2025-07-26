<script>
	import { onMount, onDestroy } from "svelte";

	// Constants for animation physics
	const GRAVITY = 0.5; // Pixels per frame squared
	const BOUNCE_DAMPING = 0.7; // How much velocity is retained after a bounce (0-1)
	const MIN_BOUNCE_VELOCITY = 0.8; // Minimum velocity to trigger another bounce
	const FINAL_Y_OFFSET = 20; // How far from the bottom the dice settle

	// Reactive state variables for Svelte
	let canvas; // Reference to the canvas DOM element
	let ctx; // 2D rendering context of the canvas

	// Dice array: Each die now has velocity (vy) and a dropping state
	let dice = $state([
		{
			x: 0,
			y: 0,
			value: 1,
			targetValue: 1,
			isRolling: false,
			isDropping: false,
			vy: 0,
		},
		{
			x: 0,
			y: 0,
			value: 1,
			targetValue: 1,
			isRolling: false,
			isDropping: false,
			vy: 0,
		},
	]);
	let isAnyDieAnimating = $state(false); // Tracks if any die is currently rolling or dropping
	let rollStartTime = $state(0);
	const rollDuration = 1000; // milliseconds for the rapid face change part of the roll

	// Function to get a random integer between min and max (inclusive)
	function getRandomInt(min, max) {
		min = Math.ceil(min);
		max = Math.floor(max);
		return Math.floor(Math.random() * (max - min + 1)) + min;
	}

	// Function to draw a rounded rectangle (for the die face)
	function roundRect(ctx, x, y, width, height, radius) {
		if (width < 2 * radius) radius = width / 2;
		if (height < 2 * radius) radius = height / 2;
		ctx.beginPath();
		ctx.moveTo(x + radius, y);
		ctx.arcTo(x + width, y, x + width, y + height, radius);
		ctx.arcTo(x + width, y + height, x, y + height, radius);
		ctx.arcTo(x, y + height, x, y, radius);
		ctx.arcTo(x, y, x + width, y, radius);
		ctx.closePath();
	}

	// Function to draw a single die face with pips (dots)
	function drawDie(ctx, x, y, size, value) {
		const pipSize = size / 8; // Size of the dots
		const offset = size / 4; // Offset for dot positions

		// Draw the die background (rounded rectangle)
		ctx.fillStyle = "#f0f0f0"; // Light grey for the die face
		roundRect(ctx, x, y, size, size, size / 8); // Rounded corners
		ctx.fill();
		ctx.strokeStyle = "#333"; // Darker border
		ctx.lineWidth = 2;
		roundRect(ctx, x, y, size, size, size / 8);
		ctx.stroke();

		ctx.fillStyle = "#333"; // Color for the pips

		// Function to draw a pip at a relative position
		const drawPip = (px, py) => {
			ctx.beginPath();
			ctx.arc(x + px, y + py, pipSize, 0, Math.PI * 2);
			ctx.fill();
		};

		// Draw pips based on the die value
		switch (value) {
			case 1:
				drawPip(size / 2, size / 2); // Center
				break;
			case 2:
				drawPip(offset, offset); // Top-left
				drawPip(size - offset, size - offset); // Bottom-right
				break;
			case 3:
				drawPip(offset, offset); // Top-left
				drawPip(size / 2, size / 2); // Center
				drawPip(size - offset, size - offset); // Bottom-right
				break;
			case 4:
				drawPip(offset, offset); // Top-left
				drawPip(size - offset, offset); // Top-right
				drawPip(offset, size - offset); // Bottom-left
				drawPip(size - offset, size - offset); // Bottom-right
				break;
			case 5:
				drawPip(offset, offset); // Top-left
				drawPip(size - offset, offset); // Top-right
				drawPip(offset, size - offset); // Bottom-left
				drawPip(size - offset, size - offset); // Bottom-right
				drawPip(size / 2, size / 2); // Center
				break;
			case 6:
				drawPip(offset, offset); // Top-left
				drawPip(size - offset, offset); // Top-right
				drawPip(offset, size / 2); // Middle-left
				drawPip(size - offset, size / 2); // Middle-right
				drawPip(offset, size - offset); // Bottom-left
				drawPip(size - offset, size - offset); // Bottom-right
				break;
			default:
				// Handle invalid value or show a blank die
				break;
		}
	}

	// Main animation loop
	function animate(timestamp) {
		if (!rollStartTime) rollStartTime = timestamp;
		const elapsed = timestamp - rollStartTime;

		// Clear the entire canvas
		ctx.clearRect(0, 0, canvas.width, canvas.height);

		const dieSize =
			Math.min(canvas.width / 3, canvas.height / 2) - FINAL_Y_OFFSET; // Adjust die size dynamically, accounting for final Y offset
		const padding = 20;
		const totalDiceWidth = dice.length * dieSize + (dice.length - 1) * padding;
		const startX = (canvas.width - totalDiceWidth) / 2;
		const finalY = canvas.height - dieSize - FINAL_Y_OFFSET; // Target Y position for settled dice

		isAnyDieAnimating = false; // Assume no dice are animating until proven otherwise

		dice.forEach((die, index) => {
			// Update horizontal position (remains centered)
			die.x = startX + index * (dieSize + padding);

			if (elapsed < rollDuration) {
				// Rolling phase: display random numbers rapidly and start falling
				die.value = getRandomInt(1, 6);
				die.isRolling = true;
				die.isDropping = true; // Start dropping immediately
				isAnyDieAnimating = true;
			} else if (die.isDropping) {
				// Dropping/Bouncing phase
				die.isRolling = false; // Stop rapid face change after rollDuration
				die.vy += GRAVITY; // Apply gravity
				die.y += die.vy; // Update vertical position

				// Collision with bottom boundary
				if (die.y >= finalY) {
					die.y = finalY; // Clamp to the final resting position
					die.vy = -die.vy * BOUNCE_DAMPING; // Reverse and damp velocity for bounce

					// If velocity is very small, stop bouncing and settle
					if (Math.abs(die.vy) < MIN_BOUNCE_VELOCITY) {
						die.vy = 0;
						die.isDropping = false; // Die has settled
						die.value = die.targetValue; // Set to final target value
					}
				}
				isAnyDieAnimating = true; // Still animating if dropping
			} else {
				// Settled state: ensure final value and position
				die.value = die.targetValue;
				die.y = finalY;
			}
			drawDie(ctx, die.x, die.y, dieSize, die.value);
		});

		// Continue animation loop if any die is still animating
		if (isAnyDieAnimating) {
			rollAnimationFrame = requestAnimationFrame(animate);
		} else {
			// All dice have settled, reset for next roll
			rollStartTime = 0;
			rollAnimationFrame = null; // Clear the animation frame ID
		}
	}

	let rollAnimationFrame = null; // To store the requestAnimationFrame ID

	// Function to initiate the dice roll
	function rollDice() {
		if (isAnyDieAnimating) return; // Prevent multiple rolls while animating

		// Assign random target values and reset animation states for each die
		dice = dice.map((die) => ({
			...die,
			targetValue: getRandomInt(1, 6),
			isRolling: true,
			isDropping: true,
			y: -Math.random() * 100 - 50, // Start randomly above the canvas
			vy: 0, // Initial vertical velocity
		}));

		rollStartTime = 0; // Reset start time for a new roll
		isAnyDieAnimating = true;
		rollAnimationFrame = requestAnimationFrame(animate); // Start the animation
	}

	// Function to handle canvas resizing
	function resizeCanvas() {
		if (canvas) {
			canvas.width = canvas.offsetWidth;
			canvas.height = canvas.offsetHeight;
			// Redraw the dice in their current state after resize if not animating
			if (!isAnyDieAnimating && ctx) {
				ctx.clearRect(0, 0, canvas.width, canvas.height);
				const dieSize =
					Math.min(canvas.width / 3, canvas.height / 2) - FINAL_Y_OFFSET;
				const padding = 20;
				const totalDiceWidth =
					dice.length * dieSize + (dice.length - 1) * padding;
				const startX = (canvas.width - totalDiceWidth) / 2;
				const finalY = canvas.height - dieSize - FINAL_Y_OFFSET;

				dice.forEach((die, index) => {
					die.x = startX + index * (dieSize + padding);
					die.y = finalY; // Ensure settled dice are redrawn at final Y
					drawDie(ctx, die.x, die.y, dieSize, die.value);
				});
			}
		}
	}

	// Svelte lifecycle hook: runs when the component is mounted
	onMount(() => {
		ctx = canvas.getContext("2d");
		resizeCanvas(); // Initial resize and draw
		window.addEventListener("resize", resizeCanvas);
		// Draw initial dice state (e.g., value 1)
		dice.forEach((die) => (die.value = 1));
		resizeCanvas(); // Call resize again to draw initial dice correctly
	});

	// Svelte lifecycle hook: runs when the component is unmounted
	onDestroy(() => {
		window.removeEventListener("resize", resizeCanvas);
		if (rollAnimationFrame) {
			cancelAnimationFrame(rollAnimationFrame);
		}
	});
</script>

<div
	class="min-h-screen bg-gradient-to-br from-purple-700 to-indigo-900 flex flex-col items-center justify-center p-4 font-inter text-white"
>
	<h1 class="text-4xl font-bold mb-8 text-center">Rolling Dice</h1>

	<div
		class="bg-white rounded-xl shadow-2xl p-6 md:p-8 w-full max-w-2xl flex flex-col items-center"
	>
		<!-- Canvas for drawing dice -->
		<div
			class="w-full h-64 md:h-80 bg-gray-100 rounded-lg overflow-hidden mb-6 border-4 border-gray-300"
		>
			<canvas bind:this={canvas} class="w-full h-full block"></canvas>
		</div>

		<!-- Roll Button -->
		<button
			onclick={() => {rollDice ()}}
			disabled={isAnyDieAnimating}
			class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-8 rounded-full shadow-lg transition duration-300 ease-in-out transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed text-lg tracking-wide"
		>
			{isAnyDieAnimating ? "Rolling..." : "Roll"}
		</button>

		<!-- Display Results -->
		<div
			class="mt-6 text-gray-800 text-2xl font-semibold flex items-center space-x-4"
		>
			<span>Result:</span>
			{#each dice as die}
				<span
					class="bg-gray-200 px-4 py-2 rounded-lg shadow-inner text-blue-800"
				>
					{die.value}
				</span>
			{/each}
		</div>
	</div>
</div>
