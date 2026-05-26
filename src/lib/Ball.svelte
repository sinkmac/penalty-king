<script lang="ts">
	let {
		fromX = 170,
		fromY = 260,
		x = 170,
		y = 80,
		active = false,
		saved = false
	} = $props<{
		fromX?: number;
		fromY?: number;
		x?: number;
		y?: number;
		active?: boolean;
		saved?: boolean;
	}>();

	let dx = $derived(x - fromX);
	let dy = $derived(y - fromY);
</script>

{#if active}
	<div
		class="ball-track"
		class:saved
		style={`--from-x:${fromX}px; --from-y:${fromY}px; --dx:${dx}px; --dy:${dy}px;`}
		aria-hidden="true"
	>
		<span class="trail"></span>
		<span class="ball"></span>
	</div>
{/if}

<style>
	.ball-track {
		position: absolute;
		z-index: 9;
		left: var(--from-x);
		top: var(--from-y);
		width: 14px;
		height: 14px;
		margin-left: -7px;
		margin-top: -7px;
		pointer-events: none;
		animation:
			flight 400ms cubic-bezier(0.2, 0.78, 0.22, 1) forwards,
			thunk 80ms ease-out 400ms;
	}

	.ball {
		position: absolute;
		inset: 0;
		border-radius: 50%;
		background: #f5f5f0;
		box-shadow:
			0 0 0 1px rgba(10, 10, 12, 0.18) inset,
			0 0 14px rgba(245, 245, 240, 0.34);
	}

	.trail {
		position: absolute;
		right: 7px;
		top: 6px;
		width: 52px;
		height: 2px;
		border-radius: 99px;
		background: linear-gradient(90deg, rgba(245, 245, 240, 0), rgba(245, 245, 240, 0.32));
		transform-origin: right center;
		transform: rotate(16deg);
		filter: blur(0.4px);
	}

	.ball-track.saved .trail {
		background: linear-gradient(90deg, rgba(201, 169, 97, 0), rgba(201, 169, 97, 0.3));
	}

	@keyframes flight {
		0% {
			transform: translate3d(0, 0, 0) scale(0.78);
		}
		64% {
			transform: translate3d(calc(var(--dx) * 0.76), calc(var(--dy) * 0.64), 0) scale(1.02);
		}
		100% {
			transform: translate3d(var(--dx), var(--dy), 0) scale(1);
		}
	}

	@keyframes thunk {
		0% {
			transform: translate3d(var(--dx), var(--dy), 0) scale(1);
		}
		50% {
			transform: translate3d(var(--dx), var(--dy), 0) scale(1.3);
		}
		100% {
			transform: translate3d(var(--dx), var(--dy), 0) scale(1);
		}
	}
</style>
