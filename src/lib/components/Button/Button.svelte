<script lang="ts">
	import type { Snippet } from 'svelte';

	type Variant = 'primary' | 'secondary' | 'ghost' | 'danger';
	type Size = 'sm' | 'md' | 'lg';

	let {
		variant = 'primary',
		size = 'md',
		disabled = false,
		loading = false,
		type = 'button',
		onclick,
		children
	}: {
		variant?: Variant;
		size?: Size;
		disabled?: boolean;
		loading?: boolean;
		type?: 'button' | 'submit' | 'reset';
		onclick?: (event: MouseEvent) => void;
		children?: Snippet;
	} = $props();

	const className = $derived(`ds-button ds-button--${variant} ds-button--${size}`);
</script>

<button class={className} {type} {onclick} disabled={disabled || loading}>
	{#if loading}
		<span class="ds-button__spinner" aria-hidden="true"></span>
		<span>Loading</span>
	{:else}
		{@render children?.()}
	{/if}
</button>

<style>
	.ds-button {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		font-weight: 600;
		border-radius: var(--ds-radius-pill);
		border: 1px solid transparent;
		cursor: pointer;
		transition:
			transform var(--ds-motion-fast),
			background-color var(--ds-motion-base),
			border-color var(--ds-motion-base);
	}

	.ds-button:hover:not(:disabled) {
		transform: translateY(-1px);
	}

	.ds-button:disabled {
		opacity: 0.6;
		cursor: not-allowed;
	}

	.ds-button--sm {
		padding: 0.4rem 0.85rem;
		font-size: var(--ds-text-sm);
	}

	.ds-button--md {
		padding: 0.6rem 1rem;
		font-size: var(--ds-text-sm);
	}

	.ds-button--lg {
		padding: 0.8rem 1.2rem;
		font-size: var(--ds-text-md);
	}

	.ds-button--primary {
		background: var(--ds-color-primary);
		color: #fff;
	}

	.ds-button--primary:hover:not(:disabled) {
		background: var(--ds-color-primary-hover);
	}

	.ds-button--secondary {
		background: var(--ds-color-surface);
		border-color: var(--ds-color-border);
		color: var(--ds-color-text);
	}

	.ds-button--ghost {
		background: transparent;
		border-color: var(--ds-color-border);
		color: var(--ds-color-text);
	}

	.ds-button--danger {
		background: var(--ds-color-danger);
		color: #fff;
	}

	.ds-button__spinner {
		width: 0.9rem;
		height: 0.9rem;
		border: 2px solid rgb(255 255 255 / 45%);
		border-top-color: #fff;
		border-radius: 50%;
		animation: ds-spin 0.8s linear infinite;
	}

	@keyframes ds-spin {
		to {
			transform: rotate(360deg);
		}
	}
</style>
