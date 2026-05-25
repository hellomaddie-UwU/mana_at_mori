<script lang="ts">
	import type { Snippet } from 'svelte';

	type Tone = 'neutral' | 'primary' | 'success' | 'warning' | 'danger';

	let {
		tone = 'neutral',
		children
	}: {
		tone?: Tone;
		children?: Snippet;
	} = $props();
</script>

<span class={`ds-badge ds-badge--${tone}`}>
	{@render children?.()}
</span>

<style>
	.ds-badge {
		display: inline-flex;
		align-items: center;
		padding: 0.2rem 0.55rem;
		border-radius: var(--ds-radius-pill);
		font-size: var(--ds-text-xs);
		font-weight: 600;
		border: 1px solid transparent;
	}

	.ds-badge--neutral {
		background: var(--ds-color-surface-2);
		color: var(--ds-color-text);
		border-color: var(--ds-color-border);
	}

	.ds-badge--primary {
		background: var(--ds-color-primary-soft);
		color: var(--ds-color-primary);
	}

	.ds-badge--success {
		background: color-mix(in srgb, var(--ds-color-success), white 82%);
		color: var(--ds-color-success);
	}

	.ds-badge--warning {
		background: color-mix(in srgb, var(--ds-color-warning), white 82%);
		color: var(--ds-color-warning);
	}

	.ds-badge--danger {
		background: color-mix(in srgb, var(--ds-color-danger), white 84%);
		color: var(--ds-color-danger);
	}
</style>
