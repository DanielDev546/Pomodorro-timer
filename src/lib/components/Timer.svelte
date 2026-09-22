<script>
	import TimeIntervals from './TimeIntervals.svelte';

	const BREAK_DURATION = 5 * 60;
	const SESSIONS_PER_CYCLE = 4;

	const presets = [
		{ label: '15 min', seconds: 15 * 60 },
		{ label: '25 min', seconds: 25 * 60 },
		{ label: '45 min', seconds: 45 * 60 }
	];

	let focusDuration = $state(25 * 60);
	let mode = $state('focus'); // 'focus' | 'break'
	let secondsRemaining = $state(focusDuration);
	let status = $state('idle'); // 'idle' | 'running' | 'paused'
	let sessionsCompleted = $state(0);

	let intervalId = null;

	const modeDuration = $derived(mode === 'focus' ? focusDuration : BREAK_DURATION);
	const progress = $derived(1 - secondsRemaining / modeDuration);

	const minutes = $derived(Math.floor(secondsRemaining / 60));
	const seconds = $derived(secondsRemaining % 60);
	const timeLabel = $derived(
		`${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
	);

	function tick() {
		if (secondsRemaining > 0) {
			secondsRemaining -= 1;
		} else {
			handleSessionEnd();
		}
	}

	function handleSessionEnd() {
		clearInterval(intervalId);
		status = 'idle';

		if (mode === 'focus') {
			sessionsCompleted += 1;
			mode = 'break';
			secondsRemaining = BREAK_DURATION;
		} else {
			mode = 'focus';
			secondsRemaining = focusDuration;
		}
		// TODO: sound/visual cue on session end
	}

	function start() {
		if (status === 'running') return;
		status = 'running';
		intervalId = setInterval(tick, 1000);
	}

	function pause() {
		status = 'paused';
		clearInterval(intervalId);
	}

	function reset() {
		status = 'idle';
		clearInterval(intervalId);
		secondsRemaining = modeDuration;
	}

	function toggleStartPause() {
		if (status === 'running') {
			pause();
		} else {
			start();
		}
	}

	function selectPreset(secondsValue) {
		focusDuration = secondsValue;
		if (mode === 'focus' && status === 'idle') {
			secondsRemaining = secondsValue;
		}
	}

	$effect(() => {
		return () => clearInterval(intervalId);
	});
</script>

<div class="w-full max-w-sm rounded-3xl bg-neutral-900 p-8 text-center">
	<p class="mb-6 text-xs tracking-widest text-neutral-400 uppercase">
		{mode === 'focus' ? 'Focus session' : 'Break'}
	</p>

	<TimeIntervals
		{presets}
		selectedSeconds={focusDuration}
		disabled={status !== 'idle'}
		onSelect={selectPreset}
	/>

	<!-- TODO: ring visualization goes here, driven by `progress` -->
	<div
		class="mx-auto mt-6 mb-6 flex h-56 w-56 items-center justify-center rounded-full border-8 border-neutral-800"
	>
		<div>
			<span class="text-4xl font-medium text-white">{timeLabel}</span>
			<p class="mt-1 text-xs text-neutral-500">{Math.round(progress * 100)}% done</p>
		</div>
	</div>

	<div class="mb-5 flex justify-center gap-3">
		<button
			onclick={reset}
			class="flex h-11 w-11 items-center justify-center rounded-full border border-neutral-700 text-neutral-400 hover:bg-neutral-800"
		>
			↻
		</button>
		<button
			onclick={toggleStartPause}
			class="flex h-14 w-14 items-center justify-center rounded-full bg-indigo-500 text-white"
		>
			{status === 'running' ? '⏸' : '▶'}
		</button>
		<button
			onclick={handleSessionEnd}
			class="flex h-11 w-11 items-center justify-center rounded-full border border-neutral-700 text-neutral-400 hover:bg-neutral-800"
		>
			⏭
		</button>
	</div>

	<div class="flex justify-center gap-1.5">
		{#each Array(SESSIONS_PER_CYCLE) as _, i}
			<span
				class="h-2 w-2 rounded-full {i < sessionsCompleted % SESSIONS_PER_CYCLE ||
				(sessionsCompleted > 0 &&
					sessionsCompleted % SESSIONS_PER_CYCLE === 0 &&
					i < SESSIONS_PER_CYCLE)
					? 'bg-indigo-500'
					: 'bg-neutral-700'}"
			></span>
		{/each}
	</div>
</div>
