<script lang="ts">
	import {
		Alert,
		Badge,
		Button,
		Card,
		Icon,
		Input,
		Modal,
		PomodoroDisplay,
		PomodoroSettings,
		SegmentControl,
		Select,
		Textarea
	} from '$lib';

	let sampleName = $state('Mori Resident');
	let sampleEmail = $state('resident@example.com');
	let sampleNote = $state('Please leave by the tea shelf.');
	let samplePriority = $state('normal');
	let sampleMode = $state('preview');
	let sampleFocusMinutes = $state(25);
	let sampleShortBreakMinutes = $state(5);
	let sampleLongBreakMinutes = $state(15);
	let modalOpen = $state(false);
	let lastModalCloseReason = $state('none');
	let showDismissibleAlert = $state(true);

	const priorities = [
		{ label: 'Low', value: 'low' },
		{ label: 'Normal', value: 'normal' },
		{ label: 'Urgent', value: 'urgent' }
	];

	const modes = [
		{ label: 'Preview', value: 'preview' },
		{ label: 'Edit', value: 'edit' },
		{ label: 'Audit', value: 'audit' }
	];
</script>

<div class="ds-container design-system-page">
	<header class="hero">
		<Badge tone="primary">Design System v1.2</Badge>
		<h1>Mana at Mori UI Source of Truth</h1>
		<p class="ds-muted">
			Use this page as the canonical reference for tokens, component APIs, states, and migration
			away from Bootstrap component classes.
		</p>
		<div class="hero__actions">
			<Button variant="primary">Primary Action</Button>
			<Button variant="secondary">Secondary</Button>
			<Button variant="ghost">Ghost</Button>
		</div>
	</header>

	<section class="ds-stack section">
		<h2>Foundations</h2>
		<p class="ds-muted">Color, type, spacing, and motion primitives should be token-driven only.</p>

		<div class="ds-grid ds-grid-3">
			<div class="ds-swatch">
				<div class="ds-swatch__chip" style="background: var(--ds-color-primary);"></div>
				<div class="ds-swatch__meta">
					<strong>Primary</strong>
					<span>--ds-color-primary</span>
				</div>
			</div>
			<div class="ds-swatch">
				<div class="ds-swatch__chip" style="background: var(--ds-color-surface-2);"></div>
				<div class="ds-swatch__meta">
					<strong>Surface 2</strong>
					<span>--ds-color-surface-2</span>
				</div>
			</div>
			<div class="ds-swatch">
				<div class="ds-swatch__chip" style="background: var(--ds-color-danger);"></div>
				<div class="ds-swatch__meta">
					<strong>Danger</strong>
					<span>--ds-color-danger</span>
				</div>
			</div>
		</div>

		<Card title="Typography scale" subtitle="Keep headings and body on tokenized sizes.">
			<div class="ds-stack">
				<h1 class="demo-title-2xl">Heading 2XL</h1>
				<h2 class="demo-title-xl">Heading XL</h2>
				<p class="demo-body">
					Body text uses <span class="ds-kbd">--ds-text-md</span> and should remain readable across layouts.
				</p>
			</div>
		</Card>
	</section>

	<section class="ds-stack section">
		<h2>Components</h2>
		<p class="ds-muted">
			Every new UI feature should compose these primitives before introducing custom markup.
		</p>

		<section id="component-button" class="component-block">
			<Card title="Button" subtitle="Primary action component with variants and sizes">
				<div class="component-demo-row">
					<Button variant="primary">Save</Button>
					<Button variant="secondary">Cancel</Button>
					<Button variant="ghost">More</Button>
					<Button variant="danger">Delete</Button>
					<Button variant="primary" loading={true}>Saving</Button>
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>variant</code></td><td>primary | secondary | ghost | danger</td><td
									>primary</td
								><td><code>variant="danger"</code></td></tr
							>
							<tr
								><td><code>size</code></td><td>sm | md | lg</td><td>md</td><td
									><code>size="lg"</code></td
								></tr
							>
							<tr
								><td><code>loading</code></td><td>boolean</td><td>false</td><td
									><code>loading={true}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-badge" class="component-block">
			<Card title="Badge" subtitle="Compact status and metadata label">
				<div class="component-demo-row">
					<Badge tone="neutral">Neutral</Badge>
					<Badge tone="primary">Primary</Badge>
					<Badge tone="success">Success</Badge>
					<Badge tone="warning">Warning</Badge>
					<Badge tone="danger">Danger</Badge>
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>tone</code></td><td>neutral | primary | success | warning | danger</td
								><td>neutral</td><td><code>tone="success"</code></td></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-card" class="component-block">
			<Card title="Card" subtitle="Content container with optional header metadata">
				<p class="ds-muted">
					Cards standardize spacing, borders, and elevation across docs and product views.
				</p>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>title</code></td><td>string</td><td>undefined</td><td
									><code>title="Profile"</code></td
								></tr
							>
							<tr
								><td><code>subtitle</code></td><td>string</td><td>undefined</td><td
									><code>subtitle="Member details"</code></td
								></tr
							>
							<tr
								><td><code>padded</code></td><td>boolean</td><td>true</td><td
									><code>padded={false}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-input" class="component-block">
			<Card title="Input" subtitle="Single-line text field with shared form styles">
				<div class="ds-stack">
					<Input label="Name" bind:value={sampleName} placeholder="Your name" />
					<Input label="Email" bind:value={sampleEmail} placeholder="Email" type="email" />
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>label</code></td><td>string</td><td>required</td><td
									><code>label="Email"</code></td
								></tr
							>
							<tr
								><td><code>size</code></td><td>sm | md | lg</td><td>md</td><td
									><code>size="sm"</code></td
								></tr
							>
							<tr
								><td><code>invalid</code></td><td>boolean</td><td>false</td><td
									><code>invalid={true}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-select" class="component-block">
			<Card title="Select" subtitle="Dropdown input with consistent sizing and focus states">
				<Select label="Priority" bind:value={samplePriority} options={priorities} />
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>label</code></td><td>string</td><td>required</td><td
									><code>label="Priority"</code></td
								></tr
							>
							<tr
								><td><code>options</code></td><td>{'{ label: string; value: string }[]'}</td><td
									>[]</td
								><td><code>options={priorities}</code></td></tr
							>
							<tr
								><td><code>size</code></td><td>sm | md | lg</td><td>md</td><td
									><code>size="lg"</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-textarea" class="component-block">
			<Card title="Textarea" subtitle="Multi-line text input for richer notes">
				<Textarea label="Note" bind:value={sampleNote} rows={4} />
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>label</code></td><td>string</td><td>required</td><td
									><code>label="Notes"</code></td
								></tr
							>
							<tr
								><td><code>rows</code></td><td>number</td><td>4</td><td><code>rows={6}</code></td
								></tr
							>
							<tr
								><td><code>invalid</code></td><td>boolean</td><td>false</td><td
									><code>invalid={true}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-segment-control" class="component-block">
			<Card title="SegmentControl" subtitle="Lightweight view or mode switcher">
				<div class="ds-stack">
					<SegmentControl label="View mode" bind:value={sampleMode} options={modes} />
					<p class="ds-muted">Current mode: <strong>{sampleMode}</strong></p>
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>label</code></td><td>string</td><td>Options</td><td
									><code>label="View mode"</code></td
								></tr
							>
							<tr
								><td><code>options</code></td><td>{'{ label: string; value: string }[]'}</td><td
									>[]</td
								><td><code>options={modes}</code></td></tr
							>
							<tr
								><td><code>value</code></td><td>string (bindable)</td><td>''</td><td
									><code>bind:value={sampleMode}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-icon" class="component-block">
			<Card title="Icon" subtitle="Bootstrap Icons wrapper with DS sizing and tone props">
				<div class="component-demo-row">
					<Icon name="stars" size="sm" tone="muted" />
					<Icon name="stars" size="md" tone="primary" />
					<Icon name="stars" size="lg" tone="success" />
					<Icon name="stars" size="xl" tone="warning" />
					<Icon name="exclamation-octagon-fill" size="lg" tone="danger" />
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>name</code></td><td>string</td><td>required</td><td
									><code>name="stars"</code></td
								></tr
							>
							<tr
								><td><code>size</code></td><td>sm | md | lg | xl</td><td>md</td><td
									><code>size="lg"</code></td
								></tr
							>
							<tr
								><td><code>tone</code></td><td
									>default | muted | primary | success | warning | danger</td
								><td>default</td><td><code>tone="primary"</code></td></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-alert" class="component-block">
			<Card title="Alert" subtitle="Semantic feedback block with optional icon and dismiss action">
				<div class="ds-stack">
					<Alert variant="info" title="Informational notice"
						>Maintenance starts at 10 PM local time.</Alert
					>
					<Alert variant="success" title="Success">Reservation updated successfully.</Alert>
					<Alert variant="warning" title="Warning">Inventory is running low for matcha blend.</Alert
					>
					{#if showDismissibleAlert}
						<Alert
							variant="danger"
							title="Payment failed"
							dismissible={true}
							onDismiss={() => (showDismissibleAlert = false)}
						>
							Please re-check your billing details.
						</Alert>
					{:else}
						<Button variant="secondary" size="sm" onclick={() => (showDismissibleAlert = true)}
							>Reset dismissible alert</Button
						>
					{/if}
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>variant</code></td><td>info | success | warning | danger</td><td>info</td
								><td><code>variant="warning"</code></td></tr
							>
							<tr
								><td><code>dismissible</code></td><td>boolean</td><td>false</td><td
									><code>dismissible={true}</code></td
								></tr
							>
							<tr
								><td><code>onDismiss</code></td><td>() =&gt; void</td><td>undefined</td><td
									><code>{'onDismiss={onCloseAlert}'}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-pomodoro-display" class="component-block">
			<Card
				title="PomodoroDisplay"
				subtitle="Timer-focused status card for focus/break flows with progress and summary metrics"
			>
				<PomodoroDisplay
					modeLabel="Focus"
					modeTone="primary"
					modeDescription="Use this when a page needs a large timer readout and next-session context."
					statusLabel="Paused"
					statusTone="warning"
					formattedTime="12:34"
					progress={49}
					completedWorkSessions={3}
					nextLabel="Long Break"
					isRunning={false}
				/>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>modeLabel</code></td><td>string</td><td>required</td><td
									><code>modeLabel="Focus"</code></td
								></tr
							>
							<tr
								><td><code>modeTone</code></td><td
									>neutral | primary | success | warning | danger</td
								><td>primary</td><td><code>modeTone="warning"</code></td></tr
							>
							<tr
								><td><code>formattedTime</code></td><td>string</td><td>required</td><td
									><code>formattedTime="25:00"</code></td
								></tr
							>
							<tr
								><td><code>progress</code></td><td>number</td><td>0</td><td
									><code>progress={42}</code></td
								></tr
							>
							<tr
								><td><code>nextLabel</code></td><td>string</td><td>required</td><td
									><code>nextLabel="Short Break"</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-pomodoro-settings" class="component-block">
			<Card
				title="PomodoroSettings"
				subtitle="Compact settings card for editable focus and break durations"
			>
				<PomodoroSettings
					bind:workMinutes={sampleFocusMinutes}
					bind:shortBreakMinutes={sampleShortBreakMinutes}
					bind:longBreakMinutes={sampleLongBreakMinutes}
				/>
				<p class="ds-muted">
					Sample values: {sampleFocusMinutes}/{sampleShortBreakMinutes}/{sampleLongBreakMinutes}
					minutes
				</p>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>workMinutes</code></td><td>number (bindable)</td><td>25</td><td
									><code>{'bind:workMinutes={focusMinutes}'}</code></td
								></tr
							>
							<tr
								><td><code>shortBreakMinutes</code></td><td>number (bindable)</td><td>5</td><td
									><code>{'bind:shortBreakMinutes={shortBreakMinutes}'}</code></td
								></tr
							>
							<tr
								><td><code>longBreakMinutes</code></td><td>number (bindable)</td><td>15</td><td
									><code>{'bind:longBreakMinutes={longBreakMinutes}'}</code></td
								></tr
							>
							<tr
								><td><code>disabled</code></td><td>boolean</td><td>false</td><td
									><code>disabled={true}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>

		<section id="component-modal" class="component-block">
			<Card title="Modal" subtitle="Basic modal with backdrop and Escape close behavior">
				<div class="ds-stack">
					<div class="component-demo-row">
						<Button variant="primary" onclick={() => (modalOpen = true)}>Open modal</Button>
						<span class="ds-muted">Last close reason: <strong>{lastModalCloseReason}</strong></span>
					</div>
					<p class="ds-muted">
						Current scope: backdrop and Escape close are enabled, focus trap is not included yet.
					</p>
				</div>
				<div class="ds-api-wrap">
					<table class="ds-api-table">
						<thead>
							<tr><th>Prop</th><th>Type</th><th>Default</th><th>Example</th></tr>
						</thead>
						<tbody>
							<tr
								><td><code>open</code></td><td>boolean (bindable)</td><td>false</td><td
									><code>bind:open={modalOpen}</code></td
								></tr
							>
							<tr
								><td><code>closeOnEscape</code></td><td>boolean</td><td>true</td><td
									><code>closeOnEscape={false}</code></td
								></tr
							>
							<tr
								><td><code>closeOnBackdrop</code></td><td>boolean</td><td>true</td><td
									><code>closeOnBackdrop={false}</code></td
								></tr
							>
						</tbody>
					</table>
				</div>
			</Card>
		</section>
	</section>

	<section class="ds-stack section">
		<h2>Migration Rules</h2>
		<Card>
			<ul class="rules">
				<li>New UI should use Design System components first.</li>
				<li>Bootstrap grid and layout helpers are allowed during transition.</li>
				<li>Replace Bootstrap alert classes with Alert and Badge primitives.</li>
				<li>Use Icon wrapper instead of direct <code>bi bi-*</code> class strings in templates.</li>
				<li>Use Modal component instead of Bootstrap modal JS for new work.</li>
			</ul>
		</Card>
	</section>
</div>

<Modal
	bind:open={modalOpen}
	title="Confirm Reservation Update"
	onClose={(reason) => (lastModalCloseReason = reason)}
>
	<p>Apply this reservation change for all future recurring bookings?</p>
	<p class="ds-muted">This demo uses basic modal behavior with Escape and backdrop closing.</p>
	{#snippet footer()}
		<Button variant="ghost" onclick={() => (modalOpen = false)}>Cancel</Button>
		<Button variant="primary" onclick={() => (modalOpen = false)}>Confirm</Button>
	{/snippet}
</Modal>

<style>
	.design-system-page {
		padding-block: var(--ds-space-7);
		display: grid;
		gap: var(--ds-space-7);
	}

	.hero {
		display: grid;
		gap: var(--ds-space-3);
		animation: ds-fade-up var(--ds-motion-base) ease;
	}

	.hero h1 {
		font-size: var(--ds-text-2xl);
	}

	.hero__actions {
		display: flex;
		flex-wrap: wrap;
		gap: var(--ds-space-2);
	}

	.section {
		animation: ds-fade-up var(--ds-motion-base) ease;
	}

	.section h2 {
		font-size: var(--ds-text-xl);
	}

	.demo-title-2xl {
		font-size: var(--ds-text-2xl);
	}

	.demo-title-xl {
		font-size: var(--ds-text-xl);
	}

	.demo-body {
		font-size: var(--ds-text-md);
	}

	.component-demo-row {
		display: flex;
		flex-wrap: wrap;
		gap: var(--ds-space-2);
		align-items: center;
	}

	.rules {
		margin: 0;
		padding-left: 1.1rem;
		display: grid;
		gap: 0.45rem;
	}

	@keyframes ds-fade-up {
		from {
			opacity: 0;
			transform: translateY(8px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
</style>
