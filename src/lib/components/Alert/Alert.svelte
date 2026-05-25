<script lang="ts">
	import type { Snippet } from 'svelte';
	import Icon from '../Icon/Icon.svelte';

	type Variant = 'info' | 'success' | 'warning' | 'danger';

	let {
		variant = 'info',
		title,
		dismissible = false,
		showIcon = true,
		onDismiss,
		children
	}: {
		variant?: Variant;
		title?: string;
		dismissible?: boolean;
		showIcon?: boolean;
		onDismiss?: () => void;
		children?: Snippet;
	} = $props();

	const iconByVariant: Record<Variant, string> = {
		info: 'info-circle-fill',
		success: 'check-circle-fill',
		warning: 'exclamation-triangle-fill',
		danger: 'x-octagon-fill'
	};
</script>

<div class={`ds-alert ds-alert--${variant}`} role={variant === 'danger' ? 'alert' : 'status'}>
	{#if showIcon}
		<Icon
			name={iconByVariant[variant]}
			tone={variant === 'info' ? 'primary' : variant}
			className="ds-alert__icon"
		/>
	{/if}

	<div class="ds-alert__content">
		{#if title}
			<strong class="ds-alert__title">{title}</strong>
		{/if}
		<div class="ds-alert__text">
			{@render children?.()}
		</div>
	</div>

	{#if dismissible}
		<button type="button" class="ds-alert__dismiss" onclick={onDismiss} aria-label="Dismiss alert">
			<Icon name="x" decorative={true} />
		</button>
	{/if}
</div>

<style>
	.ds-alert {
		display: grid;
		grid-template-columns: auto 1fr auto;
		align-items: start;
		gap: var(--ds-space-2);
		padding: var(--ds-space-3);
		border-radius: var(--ds-radius-md);
		border: 1px solid;
	}

	.ds-alert--info {
		background: color-mix(in srgb, var(--ds-color-primary), white 90%);
		border-color: color-mix(in srgb, var(--ds-color-primary), white 60%);
	}

	.ds-alert--success {
		background: color-mix(in srgb, var(--ds-color-success), white 88%);
		border-color: color-mix(in srgb, var(--ds-color-success), white 58%);
	}

	.ds-alert--warning {
		background: color-mix(in srgb, var(--ds-color-warning), white 88%);
		border-color: color-mix(in srgb, var(--ds-color-warning), white 58%);
	}

	.ds-alert--danger {
		background: color-mix(in srgb, var(--ds-color-danger), white 90%);
		border-color: color-mix(in srgb, var(--ds-color-danger), white 62%);
	}

	.ds-alert__icon {
		margin-top: 0.15rem;
	}

	.ds-alert__content {
		display: grid;
		gap: 0.2rem;
	}

	.ds-alert__title {
		font-size: var(--ds-text-sm);
	}

	.ds-alert__text {
		font-size: var(--ds-text-sm);
		color: color-mix(in srgb, var(--ds-color-text), black 15%);
	}

	.ds-alert__dismiss {
		border: none;
		background: transparent;
		color: var(--ds-color-text-muted);
		padding: 0.1rem;
		border-radius: var(--ds-radius-sm);
	}

	.ds-alert__dismiss:hover {
		color: var(--ds-color-text);
	}
</style>
