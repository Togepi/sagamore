<script lang="ts">
	import { dataLink } from './mainNav';
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import { beforeNavigate } from '$app/navigation'; // Si tu utilises SvelteKit
	import { Menu } from '@lucide/svelte';

	let isDark = $state(false);

	// Si SvelteKit n'est pas disponible, utilise cette méthode alternative
	onMount(() => {
		// Récupère la préférence sauvegardée ou celle du système
		const saved = localStorage.getItem('theme');
		const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

		isDark = saved === 'dark' || (!saved && prefersDark);
		updateTheme();
	});

	function toggleTheme() {
		isDark = !isDark;
		updateTheme();
	}

	function updateTheme() {
		if (isDark) {
			document.documentElement.classList.add('dark');
			localStorage.setItem('theme', 'dark');
		} else {
			document.documentElement.classList.remove('dark');
			localStorage.setItem('theme', 'light');
		}
	}
	beforeNavigate(() => {
		menuHidden = false;
	});

	let menuHidden = $state(false);
</script>

<div class="fixed top-2 left-2 bg-white">
	<button
		onclick={() => {
			menuHidden = !menuHidden;
		}}
		class=" p-4 md:hidden"><Menu /></button
	>
</div>
<div class="w-60">
	<button
		aria-label="hide menu"
		onclick={() => {
			menuHidden = false;
		}}
		class="fixed inset-0 z-30 bg-black/50 backdrop-blur md:hidden {menuHidden
			? 'translate-x-0'
			: '-translate-x-full md:translate-x-0'}"
	></button>
	<main
		class="fixed top-0 left-0 h-dvh w-2/3 border-r border-r-neutral-200 md:w-60 dark:border-r-stone-800 {menuHidden
			? 'translate-x-0'
			: '-translate-x-full md:translate-x-0'} 0 z-50 flex flex-col justify-between overflow-y-scroll bg-stone-50 transition-all md:overflow-hidden md:bg-stone-100 dark:md:bg-stone-900"
	>
		<header class="flex items-center justify-between px-4 py-2">
			<a href="/" class="text-xl font-bold text-stone-900 dark:text-white ">Wiki Sagamore</a>

			<!-- Toggle dark mode -->
			<button
				onclick={toggleTheme}
				class="rounded-lg p-2 transition-colors hover:bg-stone-100 dark:hover:bg-stone-800"
				aria-label="Basculer le mode sombre"
			>
				{#if isDark}
					<!-- Icône soleil -->
					<svg class="h-4 w-4 text-yellow-500" fill="currentColor" viewBox="0 0 24 24">
						<path
							d="M12 2.25a.75.75 0 01.75.75v2.25a.75.75 0 01-1.5 0V3a.75.75 0 01.75-.75zM7.5 12a4.5 4.5 0 119 0 4.5 4.5 0 01-9 0zM18.894 6.166a.75.75 0 00-1.06-1.06l-1.591 1.59a.75.75 0 101.06 1.061l1.591-1.59zM21.75 12a.75.75 0 01-.75.75h-2.25a.75.75 0 010-1.5H21a.75.75 0 01.75.75zM17.834 18.894a.75.75 0 001.06-1.06l-1.59-1.591a.75.75 0 10-1.061 1.06l1.59 1.591zM12 18a.75.75 0 01.75.75V21a.75.75 0 01-1.5 0v-2.25A.75.75 0 0112 18zM7.758 17.303a.75.75 0 00-1.061-1.06l-1.591 1.59a.75.75 0 001.06 1.061l1.591-1.59zM6 12a.75.75 0 01-.75.75H3a.75.75 0 010-1.5h2.25A.75.75 0 016 12zM6.697 7.757a.75.75 0 001.06-1.06l-1.59-1.591a.75.75 0 00-1.061 1.06l1.59 1.591z"
						/>
					</svg>
				{:else}
					<!-- Icône lune -->
					<svg
						class="h-4 w-4 text-stone-700 dark:text-stone-300"
						fill="currentColor"
						viewBox="0 0 24 24"
					>
						<path
							fill-rule="evenodd"
							d="M9.528 1.718a.75.75 0 01.162.819A8.97 8.97 0 009 6a9 9 0 009 9 8.97 8.97 0 003.463-.69.75.75 0 01.981.98 10.503 10.503 0 01-9.694 6.46c-5.799 0-10.5-4.701-10.5-10.5 0-4.368 2.667-8.112 6.46-9.694a.75.75 0 01.818.162z"
							clip-rule="evenodd"
						/>
					</svg>
				{/if}
			</button>
		</header>

		<article class="scrollbar-hover pt-4 pb-32 md:overflow-y-auto">
			<nav class="px-2">
				{#each dataLink.navMain as section}
					<div class="mb-12">
						<!-- Section principale -->
						<p
							class="block rounded px-3 py-2 transition-colors hover:underline {page.url.pathname.includes(
								section.url
							)
								? '  font-bold  '
								: ' font-semibold  '}"
						>
							{section.title}
						</p>

						<!-- Sous-éléments toujours visibles -->
						{#if section.items}
							<div>
								{#each section.items as item}
									<a
										href={item.url}
										class=" block px-1 py-1.5 text-sm transition-all hover:bg-neutral-200/50
										 hover:text-black hover:no-underline dark:hover:text-white {page.url.pathname.includes(item.url)
											? '    pl-2  text-neutral-950  dark:text-neutral-100  '
											: 'text-neutral-700 dark:text-neutral-400   '}"
									>
										<span
											class="rounded p-2 {page.url.pathname.includes(item.url)
												? '   font-semibold dark:bg-neutral-700/50   '
												: '  '}"
										>
											{item.title}</span
										>
									</a>
								{/each}
							</div>
						{/if}
					</div>
				{/each}
			</nav>
		</article>

		<footer
			class="bottom-0 left-0 flex w-full flex-col items-start gap-y-2 bg-stone-100 px-4 py-2 md:absolute dark:bg-stone-900"
		>
		
			<p class="text-xs text-stone-900 dark:text-stone-400">© 2025 Wiki Sagamore</p>
		</footer>
	</main>
</div>

<style lang="text/css">
	/* Scrollbar visible uniquement au hover, sans fond ni flèches */
	.scrollbar-hover {
		/* Cacher par défaut */
		scrollbar-width: none;
		-ms-overflow-style: none;
	}

	.scrollbar-hover::-webkit-scrollbar {
		width: 0px;
		background: transparent;
	}

	/* Afficher au hover */
	.scrollbar-hover:hover {
		scrollbar-width: thin;
	}

	.scrollbar-hover:hover::-webkit-scrollbar {
		width: 6px;
		background: transparent;
	}

	.scrollbar-hover:hover::-webkit-scrollbar-track {
		background: transparent; /* Pas de fond */
	}

	.scrollbar-hover:hover::-webkit-scrollbar-thumb {
		background: rgba(0, 0, 0, 0.3);
		border-radius: 3px;
		border: none;
	}

	.scrollbar-hover:hover::-webkit-scrollbar-thumb:hover {
		background: rgba(0, 0, 0, 0.5);
	}

	/* Supprimer les flèches (boutons) */
	.scrollbar-hover::-webkit-scrollbar-button {
		display: none;
	}

	/* Version Firefox */
	.scrollbar-hover:hover {
		scrollbar-color: rgba(0, 0, 0, 0.3) transparent;
	}
</style>
