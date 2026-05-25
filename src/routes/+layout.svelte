<script lang="ts">
	import { onMount } from 'svelte';
	import favicon from '$lib/assets/favicon.svg';
	import 'bootstrap/dist/css/bootstrap.min.css';
	import 'bootstrap-icons/font/bootstrap-icons.min.css';
	import '$lib/styles/tokens.css';
	import '$lib/styles/base.css';
	import '$lib/styles/utilities.css';
	import '$lib/styles/components.css';

	let { children } = $props();

	onMount(async () => {
		await import('bootstrap/dist/js/bootstrap.bundle.min.js');
	});
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

{@render children()}
