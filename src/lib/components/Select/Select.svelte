<script lang="ts">
	type Option = { label: string; value: string };
	type SelectSize = 'sm' | 'md' | 'lg';

	let {
		label,
		options = [],
		value = $bindable(''),
		description,
		invalid = false,
		disabled = false,
		size = 'md'
	}: {
		label: string;
		options?: Option[];
		value?: string;
		description?: string;
		invalid?: boolean;
		disabled?: boolean;
		size?: SelectSize;
	} = $props();
</script>

<label class="ds-field">
	<span class="ds-field__label">{label}</span>
	<select class={`ds-select ds-select--${size}`} bind:value {disabled} aria-invalid={invalid}>
		{#each options as option (option.value)}
			<option value={option.value}>{option.label}</option>
		{/each}
	</select>
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

	.ds-select {
		width: 100%;
		border-radius: var(--ds-radius-sm);
		border: 1px solid var(--ds-color-border);
		background: var(--ds-color-surface);
		color: var(--ds-color-text);
		transition:
			border-color var(--ds-motion-base),
			box-shadow var(--ds-motion-base);
	}

	.ds-select:focus-visible {
		border-color: var(--ds-color-focus);
		box-shadow: 0 0 0 3px rgb(11 95 255 / 18%);
		outline: none;
	}

	.ds-select[aria-invalid='true'] {
		border-color: var(--ds-color-danger);
	}

	.ds-select:disabled {
		opacity: 0.65;
		cursor: not-allowed;
		background: var(--ds-color-surface-2);
	}

	.ds-select--sm {
		padding: 0.45rem 0.6rem;
		font-size: var(--ds-text-sm);
	}

	.ds-select--md {
		padding: 0.6rem 0.75rem;
		font-size: var(--ds-text-md);
	}

	.ds-select--lg {
		padding: 0.75rem 0.85rem;
		font-size: var(--ds-text-md);
	}
</style>
