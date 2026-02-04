<script lang="ts">
	import { onMount } from 'svelte';
	import { dataLink } from '$lib/components/navigation/mainNav';

	let isOpen = false;
	let searchInput = '';
	let filteredPages: Array<{ title: string; url: string; section: string }> = [];
	let selectedIndex = 0;

	// Flatten toutes les pages pour la recherche
	$: allPages = dataLink.navMain.flatMap((section) => [
		{ title: section.title, url: section.url, section: 'Section principale' },
		...section.items.map((item) => ({
			title: item.title,
			url: item.url,
			section: section.title
		}))
	]);

	// Filtre les pages selon la recherche
	$: if (searchInput) {
		filteredPages = allPages
			.filter(
				(page) =>
					page.title.toLowerCase().includes(searchInput.toLowerCase()) ||
					page.section.toLowerCase().includes(searchInput.toLowerCase())
			)
			.slice(0, 8); // Limite à 8 résultats
		selectedIndex = 0;
	} else {
		filteredPages = allPages.slice(0, 8);
	}

	function openCommand() {
		isOpen = true;
		searchInput = '';
		selectedIndex = 0;
		// Focus sur l'input après ouverture
		setTimeout(() => {
			document.getElementById('command-input')?.focus();
		}, 10);
	}

	function closeCommand() {
		isOpen = false;
		searchInput = '';
		selectedIndex = 0;
	}

	function navigateToPage(url: string) {
		// Navigation vers la page
		window.location.href = url;
		closeCommand();
	}

	function handleKeydown(e: KeyboardEvent) {
		if (!isOpen) return;

		switch (e.key) {
			case 'Escape':
				e.preventDefault();
				closeCommand();
				break;
			case 'ArrowDown':
				e.preventDefault();
				selectedIndex = Math.min(selectedIndex + 1, filteredPages.length - 1);
				break;
			case 'ArrowUp':
				e.preventDefault();
				selectedIndex = Math.max(selectedIndex - 1, 0);
				break;
			case 'Enter':
				e.preventDefault();
				if (filteredPages[selectedIndex]) {
					navigateToPage(filteredPages[selectedIndex].url);
				}
				break;
		}
	}

	function handleGlobalKeydown(e: KeyboardEvent) {
		// Ctrl+K ou Cmd+K pour ouvrir
		if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
			e.preventDefault();
			openCommand();
		}
	}

	onMount(() => {
		document.addEventListener('keydown', handleGlobalKeydown);
		return () => {
			document.removeEventListener('keydown', handleGlobalKeydown);
		};
	});
</script>

<!-- Trigger button (optionnel, peut être placé n'importe où) -->
<button
	on:click={openCommand}
	class="flex items-center gap-2 rounded-full border border-stone-300 bg-white px-3 py-2 text-sm transition-colors hover:bg-stone-50 dark:border-stone-600 dark:bg-stone-800 dark:hover:bg-stone-700"
>
	<svg class="h-4 w-4 text-stone-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
		<path
			stroke-linecap="round"
			stroke-linejoin="round"
			stroke-width="2"
			d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
		/>
	</svg>
	<span class="hidden text-stone-600 md:block dark:text-stone-300">Rechercher...</span>
	<kbd
		class="ml-auto hidden rounded bg-stone-100 px-1.5 py-0.5 text-xs text-stone-500 md:block dark:bg-stone-700"
		>⌘K</kbd
	>
</button>

<!-- Modal overlay -->
{#if isOpen}
	<div
		tabindex="0"
		class="bg-opacity-50 fixed inset-0 z-50 flex items-start justify-center bg-black/50 pt-[20vh] backdrop-blur-md"
		role="dialog"
		aria-modal="true"
		aria-label="Recherche de pages"
		on:click={closeCommand}
		on:keydown={handleKeydown}
	>
		<!-- Command palette -->
		<div
			tabindex="0"
			class="mx-4 w-full max-w-lg rounded-lg border border-stone-200 bg-white shadow-2xl dark:border-stone-700 dark:bg-stone-800"
			role="dialog"
			on:click|stopPropagation
			on:keydown={handleKeydown}
		>
			<!-- Search input -->
			<div class="flex items-center border-b border-stone-200 px-4 py-3 dark:border-stone-700">
				<svg
					class="mr-3 h-5 w-5 text-stone-500"
					fill="none"
					stroke="currentColor"
					viewBox="0 0 24 24"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
					/>
				</svg>
				<input
					id="command-input"
					bind:value={searchInput}
					placeholder="Rechercher une page..."
					class="flex-1 bg-transparent text-lg text-stone-900 placeholder-stone-500 outline-none dark:text-white"
					on:keydown={handleKeydown}
				/>
				<kbd class="ml-3 rounded bg-stone-100 px-2 py-1 text-xs text-stone-500 dark:bg-stone-700"
					>ESC</kbd
				>
			</div>

			<!-- Results -->
			<div class="max-h-96 overflow-y-auto py-2">
				{#if filteredPages.length === 0}
					<div class="px-4 py-8 text-center text-stone-500 dark:text-stone-400">
						<svg
							class="mx-auto mb-3 h-12 w-12 text-stone-300 dark:text-stone-600"
							fill="none"
							stroke="currentColor"
							viewBox="0 0 24 24"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M9.172 16.172a4 4 0 015.656 0M9 12h6m-6-4h6m2 5.291A7.962 7.962 0 0112 15c-2.34 0-4.291-1.1-5.291-2.709M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9"
							/>
						</svg>
						<p>Aucune page trouvée</p>
					</div>
				{:else}
					{#each filteredPages as page, index}
						<button
							class="flex w-full items-center px-4 py-3 text-left transition-colors hover:bg-stone-50 dark:hover:bg-stone-700 {selectedIndex ===
							index
								? 'bg-stone-100 dark:bg-stone-700'
								: ''}"
							on:click={() => navigateToPage(page.url)}
						>
							<div class="flex-1">
								<div class="font-medium text-stone-900 dark:text-white">
									{page.title}
								</div>
								<div class="text-sm text-stone-500 dark:text-stone-400">
									{page.section}
								</div>
							</div>
							<svg
								class="h-4 w-4 text-stone-400"
								fill="none"
								stroke="currentColor"
								viewBox="0 0 24 24"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M9 5l7 7-7 7"
								/>
							</svg>
						</button>
					{/each}
				{/if}
			</div>

			<!-- Footer tips -->
			<div
				class="flex items-center gap-4 border-t border-stone-200 px-4 py-3 text-xs text-stone-500 dark:border-stone-700"
			>
				<span class="flex items-center gap-1">
					<kbd class="rounded bg-stone-100 px-1.5 py-0.5 dark:bg-stone-700">↑↓</kbd>
					naviguer
				</span>
				<span class="flex items-center gap-1">
					<kbd class="rounded bg-stone-100 px-1.5 py-0.5 dark:bg-stone-700">⏎</kbd>
					sélectionner
				</span>
				<span class="flex items-center gap-1">
					<kbd class="rounded bg-stone-100 px-1.5 py-0.5 dark:bg-stone-700">esc</kbd>
					fermer
				</span>
			</div>
		</div>
	</div>
{/if}
