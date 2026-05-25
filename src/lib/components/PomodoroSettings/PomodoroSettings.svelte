<script lang="ts">
	import Card from '../Card/Card.svelte';

	let {
		workMinutes = $bindable(25),
		shortBreakMinutes = $bindable(5),
		longBreakMinutes = $bindable(15),
		disabled = false
	}: {
		workMinutes?: number;
		shortBreakMinutes?: number;
		longBreakMinutes?: number;
		disabled?: boolean;
	} = $props();

	function clampMinutes(value: number): number {
		if (!Number.isFinite(value)) {
			return 1;
		}

		return Math.min(90, Math.max(1, Math.round(value)));
	}

	function updateWorkMinutes(event: Event) {
		workMinutes = clampMinutes((event.currentTarget as HTMLInputElement).valueAsNumber);
	}

	function updateShortBreakMinutes(event: Event) {
		shortBreakMinutes = clampMinutes((event.currentTarget as HTMLInputElement).valueAsNumber);
	}

	function updateLongBreakMinutes(event: Event) {
		longBreakMinutes = clampMinutes((event.currentTarget as HTMLInputElement).valueAsNumber);
	}
</script>

<Card
	title="Session lengths"
	subtitle="Settings persist after refresh. Changes apply to the next reset or session."
>
	<div class="ds-pomodoro-settings">
		<div class="ds-pomodoro-settings__grid">
			<label class="ds-pomodoro-settings__field">
				<span class="ds-pomodoro-settings__label">Focus</span>
				<input
					class="ds-pomodoro-settings__input"
					type="number"
					min="1"
					max="90"
					step="1"
					value={workMinutes}
					oninput={updateWorkMinutes}
					{disabled}
				/>
				<span class="ds-pomodoro-settings__help">Minutes of focused work.</span>
			</label>

			<label class="ds-pomodoro-settings__field">
				<span class="ds-pomodoro-settings__label">Short break</span>
				<input
					class="ds-pomodoro-settings__input"
					type="number"
					min="1"
					max="90"
					step="1"
					value={shortBreakMinutes}
					oninput={updateShortBreakMinutes}
					{disabled}
				/>
				<span class="ds-pomodoro-settings__help">Breathing room between work blocks.</span>
			</label>

			<label class="ds-pomodoro-settings__field">
				<span class="ds-pomodoro-settings__label">Long break</span>
				<input
					class="ds-pomodoro-settings__input"
					type="number"
					min="1"
					max="90"
					step="1"
					value={longBreakMinutes}
					oninput={updateLongBreakMinutes}
					{disabled}
				/>
				<span class="ds-pomodoro-settings__help">Used after every fourth focus session.</span>
			</label>
		</div>
	</div>
</Card>
