<script lang="ts">
	let coordinate = $state({ yScroll: 0 });
	let mouseState = $state({ moveX: 0, moveY: 0 });
	let isMouseDown = $state(false);
	let loaderLoaded = $state(false);
	let isLoaded = $state(false);

	let mouseInitPoseX = 0;
	let mouseInitPoseY = 0;

	let velocityX = 0;
	let velocityY = 0;

	let lastX = 0;
	let lastY = 0;

	let animFrame: number | null = null;

	console.log(coordinate.yScroll);

	let scaleMap = $derived(0.3 - coordinate.yScroll / 2800);
	function updatePosition(e: MouseEvent) {
		// déplacement instantané
		const dx = e.clientX - mouseInitPoseX;
		const dy = e.clientY - mouseInitPoseY;

		mouseState.moveX += dx;
		mouseState.moveY += dy;

		// vitesse (différence entre deux frames)
		velocityX = mouseState.moveX - lastX;
		velocityY = mouseState.moveY - lastY;

		lastX = mouseState.moveX;
		lastY = mouseState.moveY;

		mouseInitPoseX = e.clientX;
		mouseInitPoseY = e.clientY;
	}
	function onWheel(e: MouseEvent) {
		console.log(e.clientX);
	}
	function onMouseDown(e: MouseEvent) {
		e.preventDefault();
		isMouseDown = true;

		mouseInitPoseX = e.clientX;
		mouseInitPoseY = e.clientY;

		// reset inertie
		if (animFrame) cancelAnimationFrame(animFrame);

		window.addEventListener('mousemove', updatePosition);
		window.addEventListener('mouseup', onMouseUp);
	}

	function onMouseUp() {
		isMouseDown = false;

		window.removeEventListener('mousemove', updatePosition);
		window.removeEventListener('mouseup', onMouseUp);
		inertia();
	}

	function inertia() {
		velocityX *= 0.92;
		velocityY *= 0.92;

		mouseState.moveX += velocityX;
		mouseState.moveY += velocityY;

		if (Math.abs(velocityX) < 0.1 && Math.abs(velocityY) < 0.1) {
			velocityX = 0;
			velocityY = 0;
			return;
		}

		animFrame = requestAnimationFrame(inertia);
	}
</script>

<svelte:window bind:scrollY={coordinate.yScroll} />
<main class="  h-400 w-dvw">
	{#if !isLoaded}
		<section class="fixed z-50 flex h-dvh w-full bg-white text-center">
			<div class="m-auto text-center">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="m-auto size-6 animate-spin"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="m20.893 13.393-1.135-1.135a2.252 2.252 0 0 1-.421-.585l-1.08-2.16a.414.414 0 0 0-.663-.107.827.827 0 0 1-.812.21l-1.273-.363a.89.89 0 0 0-.738 1.595l.587.39c.59.395.674 1.23.172 1.732l-.2.2c-.212.212-.33.498-.33.796v.41c0 .409-.11.809-.32 1.158l-1.315 2.191a2.11 2.11 0 0 1-1.81 1.025 1.055 1.055 0 0 1-1.055-1.055v-1.172c0-.92-.56-1.747-1.414-2.089l-.655-.261a2.25 2.25 0 0 1-1.383-2.46l.007-.042a2.25 2.25 0 0 1 .29-.787l.09-.15a2.25 2.25 0 0 1 2.37-1.048l1.178.236a1.125 1.125 0 0 0 1.302-.795l.208-.73a1.125 1.125 0 0 0-.578-1.315l-.665-.332-.091.091a2.25 2.25 0 0 1-1.591.659h-.18c-.249 0-.487.1-.662.274a.931.931 0 0 1-1.458-1.137l1.411-2.353a2.25 2.25 0 0 0 .286-.76m11.928 9.869A9 9 0 0 0 8.965 3.525m11.928 9.868A9 9 0 1 1 8.965 3.525"
					/>
				</svg>

				<p>Chargement de la carte...</p>
			</div>
		</section>
	{/if}

	<div class="fixed h-screen w-screen">
		<button
			onmousedown={onMouseDown}
			onwheel={onWheel}
			style="transform: scale({scaleMap}) translate({mouseState.moveX * 2}px, {mouseState.moveY *
				2}px)"
			class="relative h-full w-full {isMouseDown ? ' cursor-grabbing' : 'cursor-grab'}"
		>
			<img
				draggable="false"
				onload={() => (isLoaded = true)}
				loading="lazy"
				style="max-width: none "
				src="/img/gn_sagamore_map.avif"
				alt="La carte de Sagamore"
				class=" absolute top-1/2 left-1/2 -translate-1/2"
			/>
		</button>
	</div>
</main>
<div id="debut">aaa</div>
