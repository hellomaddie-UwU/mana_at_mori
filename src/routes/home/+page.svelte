<script lang="ts">
	import { browser } from '$app/environment';
	import {
		Alert,
		Button,
		Card,
		Icon,
		Input,
		PomodoroDisplay,
		PomodoroSettings,
		SegmentControl,
		Select,
		Textarea
	} from '$lib';
	import { flip } from 'svelte/animate';
	import { dndzone } from 'svelte-dnd-action';
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
	type ChecklistItem = {
		id: string;
		title: string;
		notes: string;
		completed: boolean;
	};
	type ProjectChecklist = {
		id: string;
		title: string;
		items: ChecklistItem[];
		settings: PomodoroSettingsState;
	};
	type PersistedProjectsState = {
		version: 1;
		projects: ProjectChecklist[];
		selectedProjectId?: string;
	};

	const STORAGE_KEY = 'mana-at-mori:pomodoro-settings:v1';
	const PROJECTS_STORAGE_KEY = 'mana-at-mori:project-checklists:v1';
	const DEFAULT_PROJECT_ID = 'project-default';
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
	let handledModeSelection: PomodoroMode = 'work';
	let remainingSeconds = $state(DEFAULT_SETTINGS.workMinutes * 60);
	let isRunning = $state(false);
	let completedWorkSessions = $state(0);
	let alertState = $state<AlertState>(null);
	let liveMessage = $state('Pomodoro timer ready.');
	let audioMessage = $state('Chime will activate after your first start.');
	let hasHydrated = false;
	let timerHandle: ReturnType<typeof setInterval> | null = null;
	let audioContext: AudioContext | null = null;

	let projects = $state<ProjectChecklist[]>([
		createProjectRecord('My Project', { ...DEFAULT_SETTINGS }, [], DEFAULT_PROJECT_ID)
	]);
	let selectedProjectId = $state(DEFAULT_PROJECT_ID);
	let handledProjectSelectionId = DEFAULT_PROJECT_ID;
	let projectTitleDraft = $state('My Project');
	let projectTitleDraftProjectId = $state(DEFAULT_PROJECT_ID);
	let newProjectTitle = $state('');
	let newTaskTitle = $state('');
	let newTaskNotes = $state('');
	let editingItemId = $state<string | null>(null);
	let editingTaskTitle = $state('');
	let editingTaskNotes = $state('');

	const selectedProject = $derived.by(
		() => projects.find((project) => project.id === selectedProjectId) ?? null
	);
	const projectOptions = $derived.by(() =>
		projects.map((project) => ({
			label: project.title.trim() || 'Untitled project',
			value: project.id
		}))
	);
	const checklistStats = $derived.by(() => {
		const total = selectedProject?.items.length ?? 0;
		const completed = selectedProject?.items.filter((item) => item.completed).length ?? 0;
		return {
			total,
			completed,
			remaining: Math.max(0, total - completed)
		};
	});
	const canAddTask = $derived(newTaskTitle.trim().length > 0);
	const sessionTitle = $derived(selectedProject?.title.trim() || 'Untitled project');

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
		if (!hasHydrated || !selectedProject) {
			return;
		}

		if (selectedProject.id !== projectTitleDraftProjectId) {
			return;
		}

		const nextTitle = projectTitleDraft;
		if (selectedProject.title === nextTitle) {
			return;
		}

		updateProject(selectedProject.id, (project) => ({ ...project, title: nextTitle }));
	});

	$effect(() => {
		if (!hasHydrated) {
			return;
		}

		const project = projects.find((candidate) => candidate.id === selectedProjectId);
		if (!project) {
			return;
		}

		const nextProjectSettings: PomodoroSettingsState = {
			workMinutes: settings.workMinutes,
			shortBreakMinutes: settings.shortBreakMinutes,
			longBreakMinutes: settings.longBreakMinutes,
			preferredMode: activeMode
		};

		if (areSettingsEqual(project.settings, nextProjectSettings)) {
			return;
		}

		updateProject(selectedProjectId, (selected) => ({
			...selected,
			settings: { ...nextProjectSettings }
		}));
	});

	$effect(() => {
		const { workMinutes, shortBreakMinutes, longBreakMinutes } = settings;

		if (!hasHydrated || !browser) {
			return;
		}

		const persistedProjects: PersistedProjectsState = {
			version: 1,
			projects,
			selectedProjectId
		};

		window.localStorage.setItem(PROJECTS_STORAGE_KEY, JSON.stringify(persistedProjects));
		const selectedForPersist = projects.find((project) => project.id === selectedProjectId);
		const persistedSettings = selectedForPersist?.settings ?? {
			workMinutes,
			shortBreakMinutes,
			longBreakMinutes,
			preferredMode: activeMode
		};

		window.localStorage.setItem(
			STORAGE_KEY,
			JSON.stringify({
				version: 1,
				workMinutes: persistedSettings.workMinutes,
				shortBreakMinutes: persistedSettings.shortBreakMinutes,
				longBreakMinutes: persistedSettings.longBreakMinutes,
				preferredMode: persistedSettings.preferredMode
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

	$effect(() => {
		if (!hasHydrated || selectedProjectId === handledProjectSelectionId) {
			return;
		}

		const project = projects.find((candidate) => candidate.id === selectedProjectId);
		if (!project) {
			return;
		}

		handledProjectSelectionId = selectedProjectId;
		projectTitleDraftProjectId = project.id;
		projectTitleDraft = project.title;
		applyProjectToTimer(project, 'switch');
		editingItemId = null;
		liveMessage = `${project.title.trim() || 'Untitled project'} is now active.`;
	});

	onMount(() => {
		if (!browser) {
			return;
		}

		const legacySettings = readLegacySettings();
		const hydratedProjects = readProjectsState();

		if (hydratedProjects) {
			projects = hydratedProjects.projects;
			selectedProjectId = hydratedProjects.selectedProjectId;
		} else {
			const firstProject = createProjectRecord(
				'My Project',
				legacySettings ? { ...legacySettings } : { ...DEFAULT_SETTINGS },
				[],
				DEFAULT_PROJECT_ID
			);

			projects = [firstProject];
			selectedProjectId = firstProject.id;
		}

		const activeProject =
			projects.find((project) => project.id === selectedProjectId) ?? projects[0];
		if (activeProject) {
			selectedProjectId = activeProject.id;
			handledProjectSelectionId = activeProject.id;
			projectTitleDraftProjectId = activeProject.id;
			projectTitleDraft = activeProject.title;
			applyProjectToTimer(activeProject, 'hydrate');
		}

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

	function readLegacySettings(): PomodoroSettingsState | null {
		if (!browser) {
			return null;
		}

		const stored = window.localStorage.getItem(STORAGE_KEY);
		if (!stored) {
			return null;
		}

		try {
			const parsed = JSON.parse(stored) as Partial<PomodoroSettingsState>;
			return {
				workMinutes: sanitizeMinutes(parsed.workMinutes, DEFAULT_SETTINGS.workMinutes),
				shortBreakMinutes: sanitizeMinutes(
					parsed.shortBreakMinutes,
					DEFAULT_SETTINGS.shortBreakMinutes
				),
				longBreakMinutes: sanitizeMinutes(
					parsed.longBreakMinutes,
					DEFAULT_SETTINGS.longBreakMinutes
				),
				preferredMode: isMode(parsed.preferredMode)
					? parsed.preferredMode
					: DEFAULT_SETTINGS.preferredMode
			};
		} catch {
			window.localStorage.removeItem(STORAGE_KEY);
			return null;
		}
	}

	function readProjectsState(): { projects: ProjectChecklist[]; selectedProjectId: string } | null {
		if (!browser) {
			return null;
		}

		const stored = window.localStorage.getItem(PROJECTS_STORAGE_KEY);
		if (!stored) {
			return null;
		}

		try {
			const parsed = JSON.parse(stored) as Partial<PersistedProjectsState>;
			if (!Array.isArray(parsed.projects)) {
				return null;
			}

			const sanitizedProjects = parsed.projects
				.map((project) => sanitizeProject(project))
				.filter((project): project is ProjectChecklist => project !== null);

			if (sanitizedProjects.length === 0) {
				return null;
			}

			const selectedId =
				typeof parsed.selectedProjectId === 'string' &&
				sanitizedProjects.some((project) => project.id === parsed.selectedProjectId)
					? parsed.selectedProjectId
					: sanitizedProjects[0].id;

			return {
				projects: sanitizedProjects,
				selectedProjectId: selectedId
			};
		} catch {
			window.localStorage.removeItem(PROJECTS_STORAGE_KEY);
			return null;
		}
	}

	function sanitizeProject(value: unknown): ProjectChecklist | null {
		if (!value || typeof value !== 'object') {
			return null;
		}

		const project = value as Partial<ProjectChecklist>;
		const id =
			typeof project.id === 'string' && project.id.trim().length > 0
				? project.id
				: createId('project');
		const title = typeof project.title === 'string' ? project.title : 'Untitled project';
		const settingsCandidate = project.settings as Partial<PomodoroSettingsState> | undefined;
		const projectSettings: PomodoroSettingsState = {
			workMinutes: sanitizeMinutes(settingsCandidate?.workMinutes, DEFAULT_SETTINGS.workMinutes),
			shortBreakMinutes: sanitizeMinutes(
				settingsCandidate?.shortBreakMinutes,
				DEFAULT_SETTINGS.shortBreakMinutes
			),
			longBreakMinutes: sanitizeMinutes(
				settingsCandidate?.longBreakMinutes,
				DEFAULT_SETTINGS.longBreakMinutes
			),
			preferredMode: isMode(settingsCandidate?.preferredMode)
				? settingsCandidate.preferredMode
				: DEFAULT_SETTINGS.preferredMode
		};

		const items = Array.isArray(project.items)
			? project.items
					.map((item) => sanitizeChecklistItem(item))
					.filter((item): item is ChecklistItem => item !== null)
			: [];

		return {
			id,
			title,
			items,
			settings: projectSettings
		};
	}

	function sanitizeChecklistItem(value: unknown): ChecklistItem | null {
		if (!value || typeof value !== 'object') {
			return null;
		}

		const item = value as Partial<ChecklistItem>;
		if (typeof item.title !== 'string') {
			return null;
		}

		return {
			id: typeof item.id === 'string' && item.id.trim().length > 0 ? item.id : createId('item'),
			title: item.title,
			notes: typeof item.notes === 'string' ? item.notes : '',
			completed: Boolean(item.completed)
		};
	}

	function createProjectRecord(
		title: string,
		projectSettings: PomodoroSettingsState,
		items: ChecklistItem[] = [],
		id?: string
	): ProjectChecklist {
		return {
			id: id ?? createId('project'),
			title,
			items,
			settings: {
				workMinutes: sanitizeMinutes(projectSettings.workMinutes, DEFAULT_SETTINGS.workMinutes),
				shortBreakMinutes: sanitizeMinutes(
					projectSettings.shortBreakMinutes,
					DEFAULT_SETTINGS.shortBreakMinutes
				),
				longBreakMinutes: sanitizeMinutes(
					projectSettings.longBreakMinutes,
					DEFAULT_SETTINGS.longBreakMinutes
				),
				preferredMode: isMode(projectSettings.preferredMode)
					? projectSettings.preferredMode
					: DEFAULT_SETTINGS.preferredMode
			}
		};
	}

	function createId(prefix: string): string {
		if (browser && typeof window.crypto?.randomUUID === 'function') {
			return `${prefix}-${window.crypto.randomUUID()}`;
		}

		return `${prefix}-${Date.now()}-${Math.round(Math.random() * 1_000_000)}`;
	}

	function updateProject(
		projectId: string,
		updater: (project: ProjectChecklist) => ProjectChecklist
	) {
		projects = projects.map((project) => (project.id === projectId ? updater(project) : project));
	}

	function areSettingsEqual(left: PomodoroSettingsState, right: PomodoroSettingsState): boolean {
		return (
			left.workMinutes === right.workMinutes &&
			left.shortBreakMinutes === right.shortBreakMinutes &&
			left.longBreakMinutes === right.longBreakMinutes &&
			left.preferredMode === right.preferredMode
		);
	}

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

	function applyProjectToTimer(project: ProjectChecklist, source: 'hydrate' | 'switch') {
		settings.workMinutes = sanitizeMinutes(
			project.settings.workMinutes,
			DEFAULT_SETTINGS.workMinutes
		);
		settings.shortBreakMinutes = sanitizeMinutes(
			project.settings.shortBreakMinutes,
			DEFAULT_SETTINGS.shortBreakMinutes
		);
		settings.longBreakMinutes = sanitizeMinutes(
			project.settings.longBreakMinutes,
			DEFAULT_SETTINGS.longBreakMinutes
		);

		const preferredMode = isMode(project.settings.preferredMode)
			? project.settings.preferredMode
			: DEFAULT_SETTINGS.preferredMode;

		activeMode = preferredMode;
		modeSelection = preferredMode;
		handledModeSelection = preferredMode;
		isRunning = false;
		remainingSeconds = minutesFor(preferredMode) * 60;

		if (source === 'switch') {
			alertState = null;
		}
	}

	function switchMode(mode: PomodoroMode, source: 'manual' | 'completion') {
		activeMode = mode;
		modeSelection = mode;
		handledModeSelection = mode;
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

	function createProject() {
		const baseTitle = newProjectTitle.trim() || 'Untitled project';
		const nextProject = createProjectRecord(baseTitle, {
			workMinutes: settings.workMinutes,
			shortBreakMinutes: settings.shortBreakMinutes,
			longBreakMinutes: settings.longBreakMinutes,
			preferredMode: activeMode
		});

		projects = [...projects, nextProject];
		selectedProjectId = nextProject.id;
		handledProjectSelectionId = nextProject.id;
		projectTitleDraftProjectId = nextProject.id;
		projectTitleDraft = nextProject.title;
		applyProjectToTimer(nextProject, 'switch');
		newProjectTitle = '';
		liveMessage = `${baseTitle} created and selected.`;
	}

	function deleteSelectedProject() {
		if (!selectedProject) {
			return;
		}

		const deletedId = selectedProject.id;
		const deletedTitle = selectedProject.title.trim() || 'Untitled project';
		let remainingProjects = projects.filter((project) => project.id !== deletedId);

		if (remainingProjects.length === 0) {
			remainingProjects = [
				createProjectRecord('My Project', {
					workMinutes: settings.workMinutes,
					shortBreakMinutes: settings.shortBreakMinutes,
					longBreakMinutes: settings.longBreakMinutes,
					preferredMode: activeMode
				})
			];
		}

		projects = remainingProjects;
		const nextProject = remainingProjects[0];
		selectedProjectId = nextProject.id;
		handledProjectSelectionId = nextProject.id;
		projectTitleDraftProjectId = nextProject.id;
		projectTitleDraft = nextProject.title;
		applyProjectToTimer(nextProject, 'switch');
		editingItemId = null;
		liveMessage = `${deletedTitle} deleted. ${nextProject.title.trim() || 'Untitled project'} is now active.`;
	}

	function addChecklistItem() {
		if (!selectedProject) {
			return;
		}

		const title = newTaskTitle.trim();
		if (!title) {
			return;
		}

		const nextItem: ChecklistItem = {
			id: createId('item'),
			title,
			notes: newTaskNotes.trim(),
			completed: false
		};

		updateProject(selectedProject.id, (project) => ({
			...project,
			items: [...project.items, nextItem]
		}));
		newTaskTitle = '';
		newTaskNotes = '';
	}

	function toggleChecklistItem(itemId: string) {
		if (!selectedProject) {
			return;
		}

		updateProject(selectedProject.id, (project) => ({
			...project,
			items: project.items.map((item) =>
				item.id === itemId ? { ...item, completed: !item.completed } : item
			)
		}));
	}

	function deleteChecklistItem(itemId: string) {
		if (!selectedProject) {
			return;
		}

		updateProject(selectedProject.id, (project) => ({
			...project,
			items: project.items.filter((item) => item.id !== itemId)
		}));

		if (editingItemId === itemId) {
			cancelItemEdit();
		}
	}

	function startItemEdit(item: ChecklistItem) {
		editingItemId = item.id;
		editingTaskTitle = item.title;
		editingTaskNotes = item.notes;
	}

	function saveItemEdit(itemId: string) {
		if (!selectedProject) {
			return;
		}

		const trimmedTitle = editingTaskTitle.trim();
		if (!trimmedTitle) {
			return;
		}

		updateProject(selectedProject.id, (project) => ({
			...project,
			items: project.items.map((item) =>
				item.id === itemId ? { ...item, title: trimmedTitle, notes: editingTaskNotes.trim() } : item
			)
		}));
		cancelItemEdit();
	}

	function cancelItemEdit() {
		editingItemId = null;
		editingTaskTitle = '';
		editingTaskNotes = '';
	}

	function moveChecklistItem(itemId: string, direction: 'up' | 'down') {
		if (!selectedProject) {
			return;
		}

		const index = selectedProject.items.findIndex((item) => item.id === itemId);
		if (index < 0) {
			return;
		}

		const targetIndex = direction === 'up' ? index - 1 : index + 1;
		if (targetIndex < 0 || targetIndex >= selectedProject.items.length) {
			return;
		}

		const nextItems = [...selectedProject.items];
		const [moved] = nextItems.splice(index, 1);
		nextItems.splice(targetIndex, 0, moved);

		updateProject(selectedProject.id, (project) => ({
			...project,
			items: nextItems
		}));
	}

	function handleChecklistReorder(event: CustomEvent<{ items: ChecklistItem[] }>) {
		if (!selectedProject) {
			return;
		}

		const reorderedItems = event.detail.items.map((item) => ({
			id: item.id,
			title: item.title,
			notes: item.notes,
			completed: Boolean(item.completed)
		}));

		updateProject(selectedProject.id, (project) => ({
			...project,
			items: reorderedItems
		}));
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
			Pick a project, plan your checklist, and run focused sessions without leaving this page.
		</p>
	</header>

	<div class="pomodoro-page__layout">
		<section class="pomodoro-page__main ds-stack" aria-label="Pomodoro timer controls">
			<SegmentControl label="Pomodoro mode" options={MODE_OPTIONS} bind:value={modeSelection} />

			<PomodoroDisplay
				{sessionTitle}
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
			<Card
				title="Projects"
				subtitle="Switch projects to change both checklist and timer settings."
			>
				<div class="pomodoro-page__project-controls">
					<Select
						label="Saved projects"
						options={projectOptions}
						bind:value={selectedProjectId}
						description="Switching projects updates the timer session title immediately."
					/>

					<Input
						label="Project title"
						placeholder="Example: Website launch"
						bind:value={projectTitleDraft}
						description="This becomes the current Pomodoro session title."
					/>

					<div class="pomodoro-page__project-create">
						<Input
							label="New project"
							placeholder="Add another project"
							bind:value={newProjectTitle}
						/>
						<Button variant="secondary" onclick={createProject}>
							<Icon name="plus-lg" decorative={true} />
							Create
						</Button>
					</div>

					<Button variant="danger" onclick={deleteSelectedProject}>
						<Icon name="trash" decorative={true} />
						Delete current project
					</Button>
				</div>
			</Card>

			<Card
				title="Checklist"
				subtitle="Drag to reorder on desktop and mobile. Use arrows for keyboard-friendly reorder."
			>
				<div class="pomodoro-page__checklist-header">
					<span>{checklistStats.total} total</span>
					<span>{checklistStats.completed} completed</span>
					<span>{checklistStats.remaining} remaining</span>
				</div>

				<div class="pomodoro-page__task-create">
					<Input label="Task" placeholder="What needs to get done?" bind:value={newTaskTitle} />
					<Textarea
						label="Notes"
						placeholder="Optional details for this task"
						rows={3}
						bind:value={newTaskNotes}
					/>
					<Button variant="secondary" onclick={addChecklistItem} disabled={!canAddTask}>
						<Icon name="plus-lg" decorative={true} />
						Add task
					</Button>
				</div>

				{#if selectedProject && selectedProject.items.length > 0}
					<ul
						class="pomodoro-page__checklist"
						use:dndzone={{ items: selectedProject.items, flipDurationMs: 180 }}
						onconsider={handleChecklistReorder}
						onfinalize={handleChecklistReorder}
					>
						{#each selectedProject.items as item, index (item.id)}
							<li class="pomodoro-page__task" animate:flip={{ duration: 180 }}>
								<div class="pomodoro-page__task-top">
									<span class="pomodoro-page__task-drag" aria-hidden="true">
										<Icon name="grip-vertical" decorative={true} tone="muted" />
									</span>
									<input
										class="pomodoro-page__task-toggle"
										type="checkbox"
										checked={item.completed}
										onchange={() => toggleChecklistItem(item.id)}
										aria-label={`Toggle completion for ${item.title}`}
									/>

									{#if editingItemId === item.id}
										<div class="pomodoro-page__task-edit-fields">
											<Input label="Task title" bind:value={editingTaskTitle} />
											<Textarea label="Task notes" rows={2} bind:value={editingTaskNotes} />
										</div>
									{:else}
										<div class="pomodoro-page__task-copy">
											<p class={`pomodoro-page__task-title ${item.completed ? 'is-complete' : ''}`}>
												{item.title}
											</p>
											{#if item.notes}
												<p class="pomodoro-page__task-notes">{item.notes}</p>
											{/if}
										</div>
									{/if}
								</div>

								<div class="pomodoro-page__task-actions">
									<Button
										variant="ghost"
										size="sm"
										onclick={() => moveChecklistItem(item.id, 'up')}
										disabled={index === 0}
									>
										<Icon name="arrow-up" decorative={true} />
										Move up
									</Button>
									<Button
										variant="ghost"
										size="sm"
										onclick={() => moveChecklistItem(item.id, 'down')}
										disabled={index === selectedProject.items.length - 1}
									>
										<Icon name="arrow-down" decorative={true} />
										Move down
									</Button>

									{#if editingItemId === item.id}
										<Button variant="secondary" size="sm" onclick={() => saveItemEdit(item.id)}>
											<Icon name="check-lg" decorative={true} />
											Save
										</Button>
										<Button variant="ghost" size="sm" onclick={cancelItemEdit}>
											<Icon name="x-lg" decorative={true} />
											Cancel
										</Button>
									{:else}
										<Button variant="ghost" size="sm" onclick={() => startItemEdit(item)}>
											<Icon name="pencil" decorative={true} />
											Edit
										</Button>
										<Button variant="ghost" size="sm" onclick={() => deleteChecklistItem(item.id)}>
											<Icon name="trash" decorative={true} />
											Delete
										</Button>
									{/if}
								</div>
							</li>
						{/each}
					</ul>
				{:else}
					<p class="pomodoro-page__empty ds-muted">
						No tasks yet. Add one to start building your sprint.
					</p>
				{/if}
			</Card>

			<PomodoroSettings
				bind:workMinutes={settings.workMinutes}
				bind:shortBreakMinutes={settings.shortBreakMinutes}
				bind:longBreakMinutes={settings.longBreakMinutes}
				disabled={false}
			/>

			<Card title="Session rules" subtitle="Behavior fixed for this release.">
				<ul class="pomodoro-page__rules">
					<li>Completing focus switches to the next break, but it does not auto-start.</li>
					<li>Completing any break returns the timer to focus mode.</li>
					<li>Project title, checklist, and timer settings persist after refresh.</li>
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
		grid-template-columns: minmax(0, 1.7fr) minmax(20rem, 1fr);
		gap: var(--ds-space-4);
		align-items: start;
	}

	.pomodoro-page__controls {
		display: flex;
		flex-wrap: wrap;
		gap: var(--ds-space-3);
	}

	.pomodoro-page__project-controls,
	.pomodoro-page__task-create {
		display: grid;
		gap: var(--ds-space-3);
	}

	.pomodoro-page__project-create {
		display: grid;
		grid-template-columns: minmax(0, 1fr) auto;
		gap: var(--ds-space-2);
		align-items: end;
	}

	.pomodoro-page__checklist-header {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: var(--ds-space-2);
		margin-bottom: var(--ds-space-3);
		font-size: var(--ds-text-xs);
		text-transform: uppercase;
		letter-spacing: 0.05em;
		font-weight: 700;
		color: var(--ds-color-text-muted);
	}

	.pomodoro-page__checklist {
		list-style: none;
		margin: var(--ds-space-3) 0 0;
		padding: 0;
		display: grid;
		gap: var(--ds-space-3);
	}

	.pomodoro-page__task {
		border: 1px solid var(--ds-color-border);
		border-radius: var(--ds-radius-md);
		background: var(--ds-color-surface-2);
		padding: var(--ds-space-3);
		display: grid;
		gap: var(--ds-space-3);
	}

	.pomodoro-page__task-top {
		display: grid;
		grid-template-columns: auto auto minmax(0, 1fr);
		gap: var(--ds-space-2);
		align-items: start;
	}

	.pomodoro-page__task-drag {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding-top: 0.1rem;
		cursor: grab;
	}

	.pomodoro-page__task-copy {
		display: grid;
		gap: 0.25rem;
	}

	.pomodoro-page__task-toggle {
		appearance: none;
		width: 18px;
		height: 18px;
		border: 1.5px solid var(--ds-color-border);
		border-radius: var(--ds-radius-sm);
		background: var(--ds-color-surface);
		cursor: pointer;
		position: relative;
		margin-top: 0.15rem;
		transition:
			border-color var(--ds-motion-base),
			background-color var(--ds-motion-base),
			box-shadow var(--ds-motion-base);
	}

	.pomodoro-page__task-toggle:checked {
		background: var(--ds-color-primary);
		border-color: var(--ds-color-primary);
	}

	.pomodoro-page__task-toggle:checked::after {
		content: '';
		position: absolute;
		left: 5px;
		top: 1px;
		width: 5px;
		height: 10px;
		border: 2px solid #fff;
		border-top: none;
		border-left: none;
		transform: rotate(45deg);
	}

	.pomodoro-page__task-toggle:focus-visible {
		outline: none;
		box-shadow: 0 0 0 3px color-mix(in srgb, var(--ds-color-focus), white 75%);
	}

	.pomodoro-page__task-title {
		margin: 0;
		font-size: var(--ds-text-md);
		font-weight: 600;
	}

	.pomodoro-page__task-title.is-complete {
		text-decoration: line-through;
		color: var(--ds-color-text-muted);
	}

	.pomodoro-page__task-notes {
		margin: 0;
		font-size: var(--ds-text-sm);
		color: var(--ds-color-text-muted);
	}

	.pomodoro-page__task-edit-fields {
		display: grid;
		gap: var(--ds-space-2);
	}

	.pomodoro-page__task-actions {
		display: flex;
		flex-wrap: wrap;
		gap: var(--ds-space-2);
	}

	.pomodoro-page__empty {
		margin: var(--ds-space-3) 0 0;
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

	@media (max-width: 700px) {
		.pomodoro-page__checklist-header {
			grid-template-columns: 1fr;
		}

		.pomodoro-page__project-create {
			grid-template-columns: 1fr;
		}
	}
</style>
