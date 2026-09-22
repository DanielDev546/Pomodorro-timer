<script>
	let { presets, selectedSeconds, disabled, onSelect } = $props();
</script>

<div class="mt-4 flex items-center justify-center gap-8">
	{#each presets as preset}
		<button
			onclick={() => onSelect(preset.seconds)}
			{disabled}
			class="rounded-4xl px-4 py-1 transition-colors
				{selectedSeconds === preset.seconds
				? 'bg-indigo-500 text-white hover:bg-indigo-600'
				: 'bg-neutral-800 text-neutral-400 hover:bg-neutral-700'}
				{disabled ? 'cursor-not-allowed opacity-40' : ''}"
		>
			{preset.label}
		</button>
	{/each}
</div>
