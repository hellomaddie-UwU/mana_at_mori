<script lang="ts">
	import type { Snippet } from 'svelte';
	import Icon from '../Icon/Icon.svelte';

	type ModalSize = 'md' | 'lg';
	type CloseReason = 'backdrop' | 'escape' | 'button';

	let {
		open = $bindable(false),
		title = 'Modal',
		size = 'md',
		closeOnEscape = true,
		closeOnBackdrop = true,
		onClose,
		children,
		footer
	}: {
		open?: boolean;
		title?: string;
		size?: ModalSize;
		closeOnEscape?: boolean;
		closeOnBackdrop?: boolean;
		onClose?: (reason: CloseReason) => void;
		children?: Snippet;
		footer?: Snippet;
	} = $props();

	const headingId = `ds-modal-heading-${Math.random().toString(36).slice(2, 10)}`;

	function close(reason: CloseReason) {
		open = false;
		onClose?.(reason);
	}

	function handleBackdropClick(event: MouseEvent) {
		if (!closeOnBackdrop) {
			return;
		}
		if (event.currentTarget === event.target) {
			close('backdrop');
		}
	}

	$effect(() => {
		if (!open || !closeOnEscape) {
			return;
		}

		const onKeyDown = (event: KeyboardEvent) => {
			if (event.key === 'Escape') {
				event.preventDefault();
				close('escape');
			}
		};

		window.addEventListener('keydown', onKeyDown);
		return () => window.removeEventListener('keydown', onKeyDown);
	});
</script>

{#if open}
	<div class="ds-modal-backdrop" role="presentation" onclick={handleBackdropClick}>
		<div
			class={`ds-modal ds-modal--${size}`}
			role="dialog"
			aria-modal="true"
			aria-labelledby={headingId}
		>
			<header class="ds-modal__header">
				<h3 id={headingId}>{title}</h3>
				<button
					type="button"
					class="ds-modal__close"
					aria-label="Close modal"
					onclick={() => close('button')}
				>
					<Icon name="x-lg" />
				</button>
			</header>
			<div class="ds-modal__body">
				{@render children?.()}
			</div>
			{#if footer}
				<footer class="ds-modal__footer">
					{@render footer()}
				</footer>
			{/if}
		</div>
	</div>
{/if}

<style>
	.ds-modal-backdrop {
		position: fixed;
		inset: 0;
		background: rgb(15 20 30 / 55%);
		display: grid;
		place-items: center;
		padding: var(--ds-space-4);
		z-index: 2000;
	}

	.ds-modal {
		width: min(100%, 560px);
		background: var(--ds-color-surface);
		border-radius: var(--ds-radius-lg);
		border: 1px solid var(--ds-color-border);
		box-shadow: var(--ds-shadow-md);
		display: grid;
		max-height: min(80vh, 720px);
	}

	.ds-modal--lg {
		width: min(100%, 760px);
	}

	.ds-modal__header,
	.ds-modal__footer {
		padding: var(--ds-space-3) var(--ds-space-4);
	}

	.ds-modal__header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-bottom: 1px solid var(--ds-color-border);
	}

	.ds-modal__header h3 {
		font-size: var(--ds-text-lg);
	}

	.ds-modal__body {
		padding: var(--ds-space-4);
		overflow: auto;
		display: grid;
		gap: var(--ds-space-3);
	}

	.ds-modal__footer {
		display: flex;
		justify-content: flex-end;
		gap: var(--ds-space-2);
		border-top: 1px solid var(--ds-color-border);
	}

	.ds-modal__close {
		border: none;
		background: transparent;
		color: var(--ds-color-text-muted);
		border-radius: var(--ds-radius-sm);
		padding: 0.25rem;
	}

	.ds-modal__close:hover {
		color: var(--ds-color-text);
	}
</style>
