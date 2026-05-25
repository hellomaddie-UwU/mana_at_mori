<script lang="ts">
	import type { Snippet } from 'svelte';

	let {
		title,
		subtitle,
		children,
		padded = true
	}: {
		title?: string;
		subtitle?: string;
		children?: Snippet;
		padded?: boolean;
	} = $props();
</script>

<article class={`ds-card ${padded ? 'ds-card--padded' : ''}`}>
	{#if title}
		<header class="ds-card__header">
			<h3>{title}</h3>
			{#if subtitle}
				<p class="ds-card__subtitle">{subtitle}</p>
			{/if}
		</header>
	{/if}
	<div class="ds-card__content">
		{@render children?.()}
	</div>
</article>

<style>
	.ds-card {
		border-radius: var(--ds-radius-lg);
		border: 1px solid var(--ds-color-border);
		background: var(--ds-color-surface);
		box-shadow: var(--ds-shadow-sm);
	}

	.ds-card--padded {
		padding: var(--ds-space-4);
	}

	.ds-card__header {
		display: grid;
		gap: var(--ds-space-1);
		margin-bottom: var(--ds-space-3);
	}

	.ds-card__header h3 {
		font-size: var(--ds-text-lg);
	}

	.ds-card__subtitle {
		font-size: var(--ds-text-sm);
		color: var(--ds-color-text-muted);
	}
</style>
