<script lang="ts">
	type IconSize = 'sm' | 'md' | 'lg' | 'xl';
	type IconTone = 'default' | 'muted' | 'primary' | 'success' | 'warning' | 'danger';

	let {
		name,
		size = 'md',
		tone = 'default',
		decorative = true,
		label,
		className = ''
	}: {
		name: string;
		size?: IconSize;
		tone?: IconTone;
		decorative?: boolean;
		label?: string;
		className?: string;
	} = $props();

	const iconClasses = $derived(
		`bi bi-${name} ds-icon ds-icon--${size} ds-icon--${tone} ${className}`.trim()
	);
</script>

<i
	class={iconClasses}
	aria-hidden={decorative}
	role={decorative ? undefined : 'img'}
	aria-label={decorative ? undefined : (label ?? name)}
></i>

<style>
	.ds-icon {
		display: inline-flex;
		line-height: 1;
	}

	.ds-icon--sm {
		font-size: 0.875rem;
	}

	.ds-icon--md {
		font-size: 1rem;
	}

	.ds-icon--lg {
		font-size: 1.25rem;
	}

	.ds-icon--xl {
		font-size: 1.5rem;
	}

	.ds-icon--default {
		color: currentColor;
	}

	.ds-icon--muted {
		color: var(--ds-color-text-muted);
	}

	.ds-icon--primary {
		color: var(--ds-color-primary);
	}

	.ds-icon--success {
		color: var(--ds-color-success);
	}

	.ds-icon--warning {
		color: var(--ds-color-warning);
	}

	.ds-icon--danger {
		color: var(--ds-color-danger);
	}
</style>
