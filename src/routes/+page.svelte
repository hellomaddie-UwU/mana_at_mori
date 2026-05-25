<script lang="ts">
	import { resolve } from '$app/paths';
</script>

<main class="ds-container home">
	<h1>Mana at Mori</h1>
	<p>The design system is now the source of truth for shared UI patterns.</p>
	<p><a href={resolve('/design-system')}>Open the Design System page</a></p>
</main>

<style>
	.home {
		padding-block: var(--ds-space-7);
		display: grid;
		gap: var(--ds-space-3);
	}
</style>
