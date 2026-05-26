<script lang="ts">
	let { result = '', points = '', muted = false } = $props<{
		result?: string;
		points?: string;
		muted?: boolean;
	}>();
</script>

{#if result}
	<div class="result" class:muted aria-live="polite">
		<div class="main">{result}</div>
		{#if points}
			<div class="points">{points}</div>
		{/if}
	</div>
{/if}

<style>
	.result {
		min-height: 86px;
		text-align: center;
		letter-spacing: 0.06em;
		text-transform: uppercase;
	}

	.main {
		font-family: 'Cormorant Garamond', Georgia, serif;
		font-size: clamp(2.7rem, 14vw, 5.25rem);
		font-weight: 400;
		line-height: 0.9;
		color: #c9a961;
		animation: mainIn 420ms ease-out both;
	}

	.points {
		margin-top: 0.72rem;
		font-family: 'Inter', system-ui, sans-serif;
		font-variant-numeric: tabular-nums;
		font-size: 0.92rem;
		font-weight: 300;
		letter-spacing: 0.18em;
		color: #f5f5f0;
		animation: pointsIn 360ms ease-out 150ms both;
	}

	.result.muted .main {
		color: rgba(245, 245, 240, 0.48);
	}

	.result.muted .points {
		color: rgba(245, 245, 240, 0.42);
	}

	@keyframes mainIn {
		from {
			opacity: 0;
			transform: translateY(7px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@keyframes pointsIn {
		from {
			opacity: 0;
			transform: translateY(4px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
</style>
