<script lang="ts">
	let { active = false, match = 'Scotland v Germany, 87\'' } = $props<{
		active?: boolean;
		match?: string;
	}>();
</script>

<div class:active class="banner" aria-live="polite">
	<span class="label">PENALTY</span>
	<span class="dash">—</span>
	<span class="match">{match}</span>
	<span class="live-dot" aria-hidden="true"></span>
</div>

<style>
	.banner {
		position: fixed;
		top: max(16px, env(safe-area-inset-top));
		left: 50%;
		z-index: 20;
		display: flex;
		align-items: center;
		gap: 0.48rem;
		width: min(calc(100vw - 32px), 420px);
		padding: 0.72rem 0.86rem;
		border: 1px solid rgba(245, 245, 240, 0.1);
		background: rgba(10, 10, 12, 0.78);
		backdrop-filter: blur(18px);
		color: #f5f5f0;
		font-family: 'Inter', system-ui, sans-serif;
		font-size: 0.72rem;
		font-weight: 300;
		letter-spacing: 0.115em;
		text-transform: uppercase;
		transform: translate(-50%, -140%);
		opacity: 0;
		transition:
			transform 400ms cubic-bezier(0.16, 1, 0.3, 1),
			opacity 260ms ease;
	}

	.banner.active {
		transform: translate(-50%, 0);
		opacity: 1;
	}

	.label {
		color: #f5f5f0;
		font-weight: 400;
	}

	.dash,
	.match {
		color: rgba(245, 245, 240, 0.78);
	}

	.live-dot {
		margin-left: auto;
		width: 7px;
		height: 7px;
		border-radius: 999px;
		background: #d63838;
		box-shadow: 0 0 0 0 rgba(214, 56, 56, 0.42);
		animation: pulse 1.15s ease-out infinite;
	}

	@keyframes pulse {
		0% {
			box-shadow: 0 0 0 0 rgba(214, 56, 56, 0.46);
		}
		70% {
			box-shadow: 0 0 0 8px rgba(214, 56, 56, 0);
		}
		100% {
			box-shadow: 0 0 0 0 rgba(214, 56, 56, 0);
		}
	}
</style>
