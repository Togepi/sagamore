<script lang="ts">
	import '../app.css';
	import Navigation from '$lib/components/navigation/Navigation.svelte';
	import Command from '$lib/components/search/Command.svelte';
	let { children } = $props();
</script>

<div class="max-w-480"><div class="fixed top-2 right-2"><Command /></div>
<div class="md:pl-60">
	<Navigation />
	{@render children()}
	
</div>
</div>
