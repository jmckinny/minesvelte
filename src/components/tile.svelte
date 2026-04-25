<script>
	let { value, visible, flagged } = $props();
	let backgroundColor = $derived(
		visible && value === '💣' ? 'bg-red-900' : visible ? '' : 'bg-slate-400'
	);
</script>

<div class="h-16 w-16 border-2 border-white {backgroundColor} flex items-center justify-center">
	{#if visible && value !== 0}
		<h3 class="h3">
			{value}
		</h3>
	{:else if flagged}
		<h3 class="h3">🚩</h3>
	{/if}
</div>
