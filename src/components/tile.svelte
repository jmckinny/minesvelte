<script>
	export let value;
	export let visible;
	export let flagged;

	$: backgroundColor = visible && value === '💣' ? 'bg-red-900' : visible ? '' : 'bg-slate-400';
</script>

<div class="border-2 border-white w-16 h-16 {backgroundColor} flex items-center justify-center">
	{#if visible && value !== 0}
		<h3 class="h3">
			{value}
		</h3>
	{:else if flagged}
		<h3 class="h3">🚩</h3>
	{/if}
</div>
