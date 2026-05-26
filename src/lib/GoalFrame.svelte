<svelte:options runes={true} />

<script lang="ts">
	import type { Snippet } from 'svelte';

	let {
		width = 340,
		height = 210,
		onTap = (_event: PointerEvent) => {},
		interactive = true,
		children
	} = $props<{
		width?: number;
		height?: number;
		onTap?: (event: PointerEvent) => void;
		interactive?: boolean;
		children?: Snippet;
	}>();
</script>

<button
	class="goal-wrap"
	class:interactive
	style={`--goal-w:${width}px; --goal-h:${height}px;`}
	onpointerdown={(event) => interactive && onTap(event)}
	aria-label="Penalty target area"
	disabled={!interactive}
>
	<svg viewBox="0 0 340 210" preserveAspectRatio="xMidYMid meet" aria-hidden="true">
		<defs>
			<pattern id="diagonal-net" width="18" height="18" patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
				<path d="M 0 9 L 18 9" stroke="#f5f5f0" stroke-width="1" opacity="0.08" />
			</pattern>
			<filter id="inner-depth" x="-10%" y="-10%" width="120%" height="120%">
				<feDropShadow dx="0" dy="5" stdDeviation="3" flood-color="#000000" flood-opacity="0.34" />
			</filter>
		</defs>
		<rect x="34" y="28" width="272" height="132" fill="url(#diagonal-net)" />
		<path class="depth" d="M34 28 L58 46 M306 28 L282 46 M58 46 H282 M58 46 V160 M282 46 V160" />
		<path class="frame" d="M34 160 V28 H306 V160" filter="url(#inner-depth)" />
		<path class="ground" d="M20 174 C92 162 242 162 320 174" />
	</svg>
	{#if children}
		{@render children()}
	{/if}
</button>

<style>
	.goal-wrap {
		position: relative;
		display: block;
		width: min(calc(100vw - 32px), var(--goal-w));
		height: min(54svh, var(--goal-h));
		max-height: 250px;
		min-height: 180px;
		padding: 0;
		border: 0;
		background: transparent;
		cursor: default;
		user-select: none;
		touch-action: manipulation;
	}

	.goal-wrap.interactive {
		cursor: crosshair;
	}

	.goal-wrap:disabled {
		color: inherit;
	}

	svg {
		display: block;
		width: 100%;
		height: 100%;
		overflow: visible;
	}

	.frame {
		fill: none;
		stroke: #f5f5f0;
		stroke-width: 2;
		stroke-linecap: square;
		stroke-linejoin: miter;
	}

	.depth {
		fill: none;
		stroke: rgba(245, 245, 240, 0.14);
		stroke-width: 1;
		stroke-linecap: square;
	}

	.ground {
		fill: none;
		stroke: rgba(245, 245, 240, 0.08);
		stroke-width: 1;
	}
</style>
