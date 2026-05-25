<script lang="ts">
	import { browser } from '$app/environment';
	import {
		Alert,
		Button,
		Card,
		Icon,
		PomodoroDisplay,
		PomodoroSettings,
		SegmentControl
	} from '$lib';
	import { onDestroy, onMount } from 'svelte';

	type PomodoroMode = 'work' | 'shortBreak' | 'longBreak';
	type AlertVariant = 'info' | 'success' | 'warning' | 'danger';
	type AlertState = {
		title: string;
		message: string;
		variant: AlertVariant;
	} | null;
	type PomodoroSettingsState = {
		workMinutes: number;
		shortBreakMinutes: number;
		longBreakMinutes: number;
		preferredMode: PomodoroMode;
	};

	const STORAGE_KEY = 'mana-at-mori:pomodoro-settings:v1';
	const LONG_BREAK_INTERVAL = 4;
	const DEFAULT_SETTINGS: PomodoroSettingsState = {
		workMinutes: 25,
		shortBreakMinutes: 5,
		longBreakMinutes: 15,
		preferredMode: 'work'
	};
	const MODE_OPTIONS = [
		{ label: 'Focus', value: 'work' },
		{ label: 'Short Break', value: 'shortBreak' },
		{ label: 'Long Break', value: 'longBreak' }
	] satisfies Array<{ label: string; value: PomodoroMode }>;
	const MODE_META: Record<
		PomodoroMode,
		{
			label: string;
			description: string;
			tone: 'primary' | 'success' | 'warning';
			icon: string;
		}
	> = {
		work: {
			label: 'Focus',
			description: 'Protect a quiet work block and keep context switching out of the room.',
			tone: 'primary',
			icon: 'hourglass-split'
		},
		shortBreak: {
			label: 'Short Break',
			description: 'Take a short reset, then return for the next focused sprint.',
			tone: 'success',
			icon: 'cup-hot'
		},
		longBreak: {
			label: 'Long Break',
			description: 'After four focus sessions, step away long enough to recover properly.',
			tone: 'warning',
			icon: 'moon-stars'
		}
	};

	let settings = $state<PomodoroSettingsState>({ ...DEFAULT_SETTINGS });
	let activeMode = $state<PomodoroMode>('work');
	let modeSelection = $state<PomodoroMode>('work');
	let handledModeSelection = 'work';
	let remainingSeconds = $state(DEFAULT_SETTINGS.workMinutes * 60);
	let isRunning = $state(false);
	let completedWorkSessions = $state(0);
	let alertState = $state<AlertState>(null);
	let liveMessage = $state('Pomodoro timer ready.');
	let audioMessage = $state('Chime will activate after your first start.');
	let hasHydrated = false;
	let timerHandle: ReturnType<typeof setInterval> | null = null;
	let audioContext: AudioContext | null = null;

	const currentModeMeta = $derived(MODE_META[activeMode]);
	const formattedTime = $derived(formatTime(remainingSeconds));
	const progress = $derived.by(() => {
		const totalSeconds = minutesFor(activeMode) * 60;

		if (totalSeconds <= 0) {
			return 0;
		}

		return ((totalSeconds - remainingSeconds) / totalSeconds) * 100;
	});
	const statusLabel = $derived.by(() => {
		if (isRunning) {
			return 'Running';
		}

		if (remainingSeconds === minutesFor(activeMode) * 60) {
			return 'Ready';
		}

		return 'Paused';
	});
	const statusTone = $derived.by(() => {
		if (isRunning) {
			return 'success';
		}

		if (remainingSeconds === minutesFor(activeMode) * 60) {
			return 'neutral';
		}

		return 'warning';
	});
	const nextModeLabel = $derived(MODE_META[getNextMode(activeMode, completedWorkSessions)].label);

	$effect(() => {
		if (!isRunning) {
			return;
		}

		timerHandle = setInterval(() => {
			if (remainingSeconds <= 1) {
				handleTimerCompletion();
				return;
			}

			remainingSeconds -= 1;
		}, 1000);

		return () => {
			if (timerHandle) {
				clearInterval(timerHandle);
				timerHandle = null;
			}
		};
	});

	$effect(() => {
		const { workMinutes, shortBreakMinutes, longBreakMinutes } = settings;

		if (!hasHydrated || !browser) {
			return;
		}

		window.localStorage.setItem(
			STORAGE_KEY,
			JSON.stringify({
				version: 1,
				workMinutes,
				shortBreakMinutes,
				longBreakMinutes,
				preferredMode: activeMode
			})
		);

		if (!isRunning) {
			remainingSeconds = minutesFor(activeMode) * 60;
		}
	});

	$effect(() => {
		if (modeSelection === handledModeSelection) {
			return;
		}

		handledModeSelection = modeSelection;
		switchMode(modeSelection, 'manual');
	});

	onMount(() => {
		if (!browser) {
			return;
		}

		const stored = window.localStorage.getItem(STORAGE_KEY);

		if (stored) {
			try {
				const parsed = JSON.parse(stored) as Partial<PomodoroSettingsState> & { version?: number };

				settings.workMinutes = sanitizeMinutes(parsed.workMinutes, DEFAULT_SETTINGS.workMinutes);
				settings.shortBreakMinutes = sanitizeMinutes(
					parsed.shortBreakMinutes,
					DEFAULT_SETTINGS.shortBreakMinutes
				);
				settings.longBreakMinutes = sanitizeMinutes(
					parsed.longBreakMinutes,
					DEFAULT_SETTINGS.longBreakMinutes
				);

				if (isMode(parsed.preferredMode)) {
					activeMode = parsed.preferredMode;
					modeSelection = parsed.preferredMode;
					handledModeSelection = parsed.preferredMode;
					settings.preferredMode = parsed.preferredMode;
				}
			} catch {
				window.localStorage.removeItem(STORAGE_KEY);
			}
		}

		remainingSeconds = minutesFor(activeMode) * 60;
		hasHydrated = true;
	});

	onDestroy(() => {
		if (timerHandle) {
			clearInterval(timerHandle);
		}

		if (audioContext && audioContext.state !== 'closed') {
			void audioContext.close();
		}
	});

	function isMode(value: unknown): value is PomodoroMode {
		return value === 'work' || value === 'shortBreak' || value === 'longBreak';
	}

	function sanitizeMinutes(value: unknown, fallback: number): number {
		if (typeof value !== 'number' || Number.isNaN(value)) {
			return fallback;
		}

		return Math.min(90, Math.max(1, Math.round(value)));
	}

	function minutesFor(mode: PomodoroMode): number {
		if (mode === 'work') {
			return settings.workMinutes;
		}

		if (mode === 'shortBreak') {
			return settings.shortBreakMinutes;
		}

		return settings.longBreakMinutes;
	}

	function formatTime(totalSeconds: number): string {
		const minutes = Math.floor(totalSeconds / 60)
			.toString()
			.padStart(2, '0');
		const seconds = Math.max(0, totalSeconds % 60)
			.toString()
			.padStart(2, '0');

		return `${minutes}:${seconds}`;
	}

	function getNextMode(mode: PomodoroMode, completedCount: number): PomodoroMode {
		if (mode === 'work') {
			return (completedCount + 1) % LONG_BREAK_INTERVAL === 0 ? 'longBreak' : 'shortBreak';
		}

		return 'work';
	}

	function switchMode(mode: PomodoroMode, source: 'manual' | 'completion') {
		activeMode = mode;
		modeSelection = mode;
		handledModeSelection = mode;
		settings.preferredMode = mode;
		isRunning = false;
		remainingSeconds = minutesFor(mode) * 60;

		if (source === 'manual') {
			alertState = null;
			liveMessage = `${MODE_META[mode].label} selected. Timer reset to ${formatTime(remainingSeconds)}.`;
		}
	}

	function startTimer() {
		initializeAudio();
		alertState = null;
		isRunning = true;
		liveMessage = `${MODE_META[activeMode].label} started.`;
	}

	function pauseTimer() {
		isRunning = false;
		liveMessage = `${MODE_META[activeMode].label} paused at ${formatTime(remainingSeconds)}.`;
	}

	function resetTimer() {
		isRunning = false;
		remainingSeconds = minutesFor(activeMode) * 60;
		alertState = null;
		liveMessage = `${MODE_META[activeMode].label} reset to ${formatTime(remainingSeconds)}.`;
	}

	function handleTimerCompletion() {
		const completedMode = activeMode;
		const completedLabel = MODE_META[completedMode].label;
		const nextCompletedCount =
			completedMode === 'work' ? completedWorkSessions + 1 : completedWorkSessions;
		const nextMode = getNextMode(completedMode, completedWorkSessions);
		const nextLabel = MODE_META[nextMode].label;

		completedWorkSessions = nextCompletedCount;
		playChime();
		switchMode(nextMode, 'completion');
		alertState = {
			title: `${completedLabel} complete`,
			message: `${nextLabel} is ready. Press Start when you want to begin the next session.`,
			variant: nextMode === 'work' ? 'success' : 'warning'
		};
		liveMessage = `${completedLabel} complete. ${nextLabel} ready to start.`;
	}

	function initializeAudio() {
		if (!browser || audioContext) {
			return;
		}

		const AudioContextClass =
			window.AudioContext ??
			(window as Window & { webkitAudioContext?: typeof AudioContext }).webkitAudioContext;

		if (!AudioContextClass) {
			audioMessage =
				'This browser does not expose the Web Audio API. Visual alerts will still appear.';
			return;
		}

		audioContext = new AudioContextClass();
		audioMessage = 'Chime is ready.';
	}

	function playChime() {
		if (!audioContext) {
			audioMessage = 'Chime was unavailable for this session. Visual alerts are still active.';
			return;
		}

		void audioContext
			.resume()
			.then(() => {
				const now = audioContext?.currentTime ?? 0;
				const gain = audioContext?.createGain();
				const oscillator = audioContext?.createOscillator();

				if (!gain || !oscillator || !audioContext) {
					return;
				}

				oscillator.type = 'sine';
				oscillator.frequency.setValueAtTime(880, now);
				oscillator.frequency.linearRampToValueAtTime(660, now + 0.18);
				gain.gain.setValueAtTime(0.0001, now);
				gain.gain.linearRampToValueAtTime(0.16, now + 0.02);
				gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.42);
				oscillator.connect(gain);
				gain.connect(audioContext.destination);
				oscillator.start(now);
				oscillator.stop(now + 0.42);
			})
			.catch(() => {
				audioMessage =
					'Browser audio is blocked until playback is allowed. Visual alerts are still active.';
			});
	}
</script>

<main class="ds-container pomodoro-page">
	<div class="pomodoro-page__live-region visually-hidden" aria-live="polite">{liveMessage}</div>

	<header class="pomodoro-page__hero">
		<div class="pomodoro-page__eyebrow">
			<Icon name={currentModeMeta.icon} tone={currentModeMeta.tone} />
			<span>Pomodoro timer</span>
		</div>
		<h1>Build a steady focus rhythm at home.</h1>
		<p class="ds-muted">
			Classic 25/5/15 defaults, manual start between sessions, and a long break after every four
			completed focus blocks.
		</p>
	</header>

	<div class="pomodoro-page__layout">
		<section class="pomodoro-page__main ds-stack" aria-label="Pomodoro timer controls">
			<SegmentControl label="Pomodoro mode" options={MODE_OPTIONS} bind:value={modeSelection} />

			<PomodoroDisplay
				modeLabel={currentModeMeta.label}
				modeTone={currentModeMeta.tone}
				modeDescription={currentModeMeta.description}
				{statusLabel}
				{statusTone}
				{formattedTime}
				{progress}
				{completedWorkSessions}
				nextLabel={nextModeLabel}
				{isRunning}
			/>

			<div class="pomodoro-page__controls">
				<Button variant="primary" size="lg" onclick={isRunning ? pauseTimer : startTimer}>
					<Icon name={isRunning ? 'pause-fill' : 'play-fill'} decorative={true} />
					{isRunning ? 'Pause' : 'Start'}
				</Button>
				<Button variant="secondary" size="lg" onclick={resetTimer}>
					<Icon name="arrow-counterclockwise" decorative={true} />
					Reset
				</Button>
			</div>

			{#if alertState}
				<Alert
					variant={alertState.variant}
					title={alertState.title}
					dismissible={true}
					onDismiss={() => (alertState = null)}
				>
					{alertState.message}
				</Alert>
			{/if}
		</section>

		<aside class="pomodoro-page__sidebar ds-stack">
			<PomodoroSettings
				bind:workMinutes={settings.workMinutes}
				bind:shortBreakMinutes={settings.shortBreakMinutes}
				bind:longBreakMinutes={settings.longBreakMinutes}
				disabled={false}
			/>

			<Card title="Session rules" subtitle="Behavior fixed for this first release.">
				<ul class="pomodoro-page__rules">
					<li>Completing focus switches to the next break, but it does not auto-start.</li>
					<li>Completing any break returns the timer to focus mode.</li>
					<li>Only durations and your preferred mode persist after refresh.</li>
				</ul>
				<p class="pomodoro-page__audio ds-muted">{audioMessage}</p>
			</Card>
		</aside>
	</div>
</main>

<style>
	.pomodoro-page {
		padding-block: var(--ds-space-7);
		display: grid;
		gap: var(--ds-space-5);
	}

	.pomodoro-page__hero {
		display: grid;
		gap: var(--ds-space-3);
		max-width: 44rem;
	}

	.pomodoro-page__hero h1 {
		font-size: var(--ds-text-2xl);
		line-height: var(--ds-line-tight);
	}

	.pomodoro-page__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: var(--ds-space-2);
		font-size: var(--ds-text-sm);
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.08em;
		color: var(--ds-color-text-muted);
	}

	.pomodoro-page__layout {
		display: grid;
		grid-template-columns: minmax(0, 1.7fr) minmax(18rem, 1fr);
		gap: var(--ds-space-4);
		align-items: start;
	}

	.pomodoro-page__controls {
		display: flex;
		flex-wrap: wrap;
		gap: var(--ds-space-3);
	}

	.pomodoro-page__rules {
		margin: 0;
		padding-left: 1.1rem;
		display: grid;
		gap: var(--ds-space-2);
		color: var(--ds-color-text-muted);
	}

	.pomodoro-page__audio {
		margin: 0;
	}

	@media (max-width: 900px) {
		.pomodoro-page__layout {
			grid-template-columns: 1fr;
		}
	}
</style>
