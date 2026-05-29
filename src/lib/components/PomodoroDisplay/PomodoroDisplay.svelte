<script lang="ts">
	import Badge from '../Badge/Badge.svelte';
	import Card from '../Card/Card.svelte';
	import Icon from '../Icon/Icon.svelte';

	type BadgeTone = 'neutral' | 'primary' | 'success' | 'warning' | 'danger';

	let {
		sessionTitle = 'Current session',
		modeLabel,
		modeTone = 'primary',
		modeDescription,
		statusLabel,
		statusTone = 'neutral',
		formattedTime,
		progress = 0,
		completedWorkSessions = 0,
		nextLabel,
		isRunning = false
	}: {
		sessionTitle?: string;
		modeLabel: string;
		modeTone?: BadgeTone;
		modeDescription: string;
		statusLabel: string;
		statusTone?: BadgeTone;
		formattedTime: string;
		progress?: number;
		completedWorkSessions?: number;
		nextLabel: string;
		isRunning?: boolean;
	} = $props();

	const clampedProgress = $derived(Math.max(0, Math.min(100, progress)));
</script>

<Card title={sessionTitle} subtitle="Manual-start Pomodoro flow with customizable durations.">
	<div class="ds-pomodoro-display">
		<div class="ds-pomodoro-display__header">
			<div class="ds-pomodoro-display__badges">
				<Badge tone={modeTone}>{modeLabel}</Badge>
				<Badge tone={statusTone}>{statusLabel}</Badge>
			</div>
			<div class="ds-pomodoro-display__state">
				<Icon
					name={isRunning ? 'play-fill' : 'pause-fill'}
					tone={isRunning ? 'success' : 'muted'}
				/>
				<span>{isRunning ? 'In progress' : 'Waiting to start'}</span>
			</div>
		</div>

		<p class="ds-pomodoro-display__description">{modeDescription}</p>

		<div
			class="ds-pomodoro-display__clock"
			role="timer"
			aria-live="off"
			aria-label={`${modeLabel} timer at ${formattedTime}`}
		>
			{formattedTime}
		</div>

		<div class="ds-pomodoro-display__progress" aria-hidden="true">
			<div class="ds-pomodoro-display__progress-bar" style={`width: ${clampedProgress}%`}></div>
		</div>

		<div class="ds-pomodoro-display__meta">
			<div class="ds-pomodoro-display__metric">
				<span class="ds-pomodoro-display__label">Completed focus sessions</span>
				<strong>{completedWorkSessions}</strong>
			</div>
			<div class="ds-pomodoro-display__metric">
				<span class="ds-pomodoro-display__label">Next up</span>
				<strong>{nextLabel}</strong>
			</div>
		</div>
	</div>
</Card>
