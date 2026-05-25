<script lang="ts">
	type InputSize = 'sm' | 'md' | 'lg';

	let {
		label,
		placeholder = '',
		type = 'text',
		value = $bindable(''),
		description,
		invalid = false,
		disabled = false,
		size = 'md'
	}: {
		label: string;
		placeholder?: string;
		type?: string;
		value?: string;
		description?: string;
		invalid?: boolean;
		disabled?: boolean;
		size?: InputSize;
	} = $props();
</script>

<label class="ds-field">
	<span class="ds-field__label">{label}</span>
	<input
		class={`ds-input ds-input--${size}`}
		{type}
		{placeholder}
		bind:value
		{disabled}
		aria-invalid={invalid}
	/>
	{#if description}
		<span class="ds-field__help">{description}</span>
	{/if}
</label>

<style>
	.ds-field {
		display: grid;
		gap: 0.35rem;
	}

	.ds-field__label {
		font-size: var(--ds-text-sm);
		font-weight: 600;
	}

	.ds-field__help {
		font-size: var(--ds-text-xs);
		color: var(--ds-color-text-muted);
	}

	.ds-input {
		width: 100%;
		border-radius: var(--ds-radius-sm);
		border: 1px solid var(--ds-color-border);
		background: var(--ds-color-surface);
		color: var(--ds-color-text);
		transition:
			border-color var(--ds-motion-base),
			box-shadow var(--ds-motion-base);
	}

	.ds-input::placeholder {
		color: color-mix(in srgb, var(--ds-color-text-muted), white 20%);
	}

	.ds-input:focus-visible {
		border-color: var(--ds-color-focus);
		box-shadow: 0 0 0 3px rgb(11 95 255 / 18%);
		outline: none;
	}

	.ds-input[aria-invalid='true'] {
		border-color: var(--ds-color-danger);
	}

	.ds-input:disabled {
		opacity: 0.65;
		cursor: not-allowed;
		background: var(--ds-color-surface-2);
	}

	.ds-input--sm {
		padding: 0.45rem 0.6rem;
		font-size: var(--ds-text-sm);
	}

	.ds-input--md {
		padding: 0.6rem 0.75rem;
		font-size: var(--ds-text-md);
	}

	.ds-input--lg {
		padding: 0.75rem 0.85rem;
		font-size: var(--ds-text-md);
	}
</style>
