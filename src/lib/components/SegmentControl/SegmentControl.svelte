<script lang="ts">
	type SegmentOption = { label: string; value: string };

	let {
		label = 'Options',
		options = [],
		value = $bindable('')
	}: {
		label?: string;
		options?: SegmentOption[];
		value?: string;
	} = $props();
</script>

<fieldset class="ds-segment" aria-label={label}>
	<legend class="visually-hidden">{label}</legend>
	{#each options as option (option.value)}
		<button
			type="button"
			class={`ds-segment__item ${value === option.value ? 'is-active' : ''}`}
			onclick={() => (value = option.value)}
		>
			{option.label}
		</button>
	{/each}
</fieldset>

<style>
	.ds-segment {
		margin: 0;
		padding: 0.25rem;
		border: 1px solid var(--ds-color-border);
		background: var(--ds-color-surface-2);
		display: inline-flex;
		gap: 0.25rem;
		border-radius: var(--ds-radius-pill);
	}

	.ds-segment__item {
		border: none;
		background: transparent;
		color: var(--ds-color-text-muted);
		border-radius: var(--ds-radius-pill);
		padding: 0.4rem 0.75rem;
		font-size: var(--ds-text-sm);
		font-weight: 600;
	}

	.ds-segment__item:hover {
		color: var(--ds-color-text);
	}

	.ds-segment__item.is-active {
		background: var(--ds-color-surface);
		color: var(--ds-color-text);
		box-shadow: var(--ds-shadow-sm);
	}
</style>
