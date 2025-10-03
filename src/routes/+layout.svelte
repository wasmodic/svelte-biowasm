<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.ico';
	
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>
<div class="max-w-xl mx-auto px-4">
	{@render children?.()}
</div>
