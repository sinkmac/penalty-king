<script lang="ts">
	let { x = 0, y = 0, visible = false, dimmed = false } = $props<{
		x?: number;
		y?: number;
		visible?: boolean;
		dimmed?: boolean;
	}>();
</script>

{#if visible}
	<div
		class:dimmed
		class="reticle"
		style={`left:${x}px; top:${y}px;`}
		aria-hidden="true"
	>
		<span></span>
	</div>
{/if}

<style>
	.reticle {
		position: absolute;
		z-index: 8;
		width: 30px;
		height: 30px;
		margin-left: -15px;
		margin-top: -15px;
		border: 1.5px solid #c9a961;
		border-radius: 50%;
		pointer-events: none;
		filter: drop-shadow(0 0 10px rgba(201, 169, 97, 0.22));
		transform: scale(1);
		animation: place 100ms ease-out both;
		transition:
			left 160ms cubic-bezier(0.16, 1, 0.3, 1),
			top 160ms cubic-bezier(0.16, 1, 0.3, 1),
			opacity 220ms ease,
			filter 220ms ease;
	}

	.reticle::before,
	.reticle::after,
	.reticle span::before,
	.reticle span::after {
		position: absolute;
		content: '';
		background: #c9a961;
		opacity: 0.86;
	}

	.reticle::before {
		left: 50%;
		top: -7px;
		width: 1px;
		height: 11px;
	}

	.reticle::after {
		left: 50%;
		bottom: -7px;
		width: 1px;
		height: 11px;
	}

	.reticle span::before {
		left: -7px;
		top: 50%;
		width: 11px;
		height: 1px;
	}

	.reticle span::after {
		right: -7px;
		top: 50%;
		width: 11px;
		height: 1px;
	}

	.reticle.dimmed {
		opacity: 0.48;
		filter: drop-shadow(0 0 4px rgba(201, 169, 97, 0.12));
		animation: lock 420ms ease-out both;
	}

	@keyframes place {
		from {
			transform: scale(1.2);
			opacity: 0.5;
		}
		to {
			transform: scale(1);
			opacity: 1;
		}
	}

	@keyframes lock {
		0% {
			transform: scale(1);
		}
		46% {
			transform: scale(1.11);
		}
		100% {
			transform: scale(1);
		}
	}
</style>
