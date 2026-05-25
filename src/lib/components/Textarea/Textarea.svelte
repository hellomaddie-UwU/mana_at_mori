<script lang="ts">
	type TextareaSize = 'sm' | 'md' | 'lg';

	let {
		label,
		placeholder = '',
		value = $bindable(''),
		description,
		invalid = false,
		disabled = false,
		rows = 4,
		size = 'md'
	}: {
		label: string;
		placeholder?: string;
		value?: string;
		description?: string;
		invalid?: boolean;
		disabled?: boolean;
		rows?: number;
		size?: TextareaSize;
	} = $props();
</script>

<label class="ds-field">
	<span class="ds-field__label">{label}</span>
	<textarea
		class={`ds-textarea ds-textarea--${size}`}
		{placeholder}
		bind:value
		{rows}
		{disabled}
		aria-invalid={invalid}
	></textarea>
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

	.ds-textarea {
		width: 100%;
		border-radius: var(--ds-radius-sm);
		border: 1px solid var(--ds-color-border);
		background: var(--ds-color-surface);
		color: var(--ds-color-text);
		resize: vertical;
		transition:
			border-color var(--ds-motion-base),
			box-shadow var(--ds-motion-base);
	}

	.ds-textarea:focus-visible {
		border-color: var(--ds-color-focus);
		box-shadow: 0 0 0 3px rgb(11 95 255 / 18%);
		outline: none;
	}

	.ds-textarea[aria-invalid='true'] {
		border-color: var(--ds-color-danger);
	}

	.ds-textarea:disabled {
		opacity: 0.65;
		cursor: not-allowed;
		background: var(--ds-color-surface-2);
	}

	.ds-textarea--sm {
		padding: 0.45rem 0.6rem;
		font-size: var(--ds-text-sm);
	}

	.ds-textarea--md {
		padding: 0.6rem 0.75rem;
		font-size: var(--ds-text-md);
	}

	.ds-textarea--lg {
		padding: 0.75rem 0.85rem;
		font-size: var(--ds-text-md);
	}
</style>
