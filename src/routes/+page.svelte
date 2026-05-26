<svelte:options runes={true} />

<script lang="ts">
	import { onDestroy } from 'svelte';
	import Ball from '$lib/Ball.svelte';
	import Banner from '$lib/Banner.svelte';
	import GoalFrame from '$lib/GoalFrame.svelte';
	import ResultDisplay from '$lib/ResultDisplay.svelte';
	import Reticle from '$lib/Reticle.svelte';

	type GameState = 'idle' | 'trigger' | 'aim' | 'lockout' | 'resolution';
	type TapPoint = { x: number; y: number };
	type Resolution = {
		x: number;
		y: number;
		label: string;
		points: string;
		muted: boolean;
		saved: boolean;
	};

	const matches = [
		"Scotland v Germany, 87'",
		"France v Brazil, 71'",
		"Argentina v Netherlands, 90+3'",
		"Italy v Spain, 64'",
		"Portugal v Croatia, 119'"
	];

	let gameState = $state<GameState>('idle');
	let selectedMatch = $state(matches[0]);
	let tapPoint: TapPoint | null = $state(null);
	let resolution: Resolution | null = $state(null);
	let progress = $state(1);
	let flash = $state(false);
	let round = $state(0);
	let goalEl: HTMLElement | null = $state(null);
	let goalSize = $state({ width: 340, height: 210 });

	let timers: number[] = [];
	let raf = 0;
	let aimStarted = 0;

	const isGoalVisible = $derived(gameState === 'aim' || gameState === 'lockout' || gameState === 'resolution');
	const isInteractive = $derived(gameState === 'aim');

	function setTimer(fn: () => void, ms: number) {
		const id = window.setTimeout(fn, ms);
		timers.push(id);
		return id;
	}

	function clearTimers() {
		for (const timer of timers) window.clearTimeout(timer);
		timers = [];
		if (raf) window.cancelAnimationFrame(raf);
		raf = 0;
	}

	function startRound() {
		clearTimers();
		round += 1;
		gameState = 'trigger';
		selectedMatch = matches[Math.floor(Math.random() * matches.length)];
		tapPoint = null;
		resolution = null;
		progress = 1;
		setTimer(() => startAim(), 1500);
	}

	function startAim() {
		gameState = 'aim';
		progress = 1;
		aimStarted = performance.now();
		tickProgress();
		setTimer(() => lockout(), 5000);
	}

	function tickProgress() {
		const elapsed = performance.now() - aimStarted;
		progress = Math.max(0, 1 - elapsed / 5000);
		if (gameState === 'aim' && progress > 0) {
			raf = requestAnimationFrame(tickProgress);
		}
	}

	function handleTap(event: PointerEvent) {
		if (gameState !== 'aim') return;
		const target = event.currentTarget as HTMLElement;
		const rect = target.getBoundingClientRect();
		goalSize = { width: rect.width, height: rect.height };
		tapPoint = {
			x: clamp(event.clientX - rect.left, 0, rect.width),
			y: clamp(event.clientY - rect.top, 0, rect.height)
		};
		flashScreen();
	}

	function flashScreen() {
		flash = false;
		requestAnimationFrame(() => {
			flash = true;
			setTimer(() => (flash = false), 50);
		});
	}

	function lockout() {
		if (gameState !== 'aim') return;
		gameState = 'lockout';
		progress = 1;
		if (!tapPoint) {
			updateGoalSize();
			tapPoint = { x: goalSize.width / 2, y: goalSize.height * 0.5 };
		}
		resolution = makeResolution(tapPoint, goalSize, round);
		setTimer(() => {
			gameState = 'resolution';
		}, 2000);
		setTimer(() => {
			gameState = 'idle';
			tapPoint = null;
			resolution = null;
			progress = 1;
		}, 5400);
	}

	function updateGoalSize() {
		if (!goalEl) return;
		const rect = goalEl.getBoundingClientRect();
		goalSize = { width: rect.width, height: rect.height };
	}

	function makeResolution(tap: TapPoint, size: { width: number; height: number }, roundNumber: number): Resolution {
		const roll = Math.random();
		let mode: 'spot' | 'close' | 'wide' | 'common' | 'save';
		if (roll < 0.25) mode = 'spot';
		else if (roll < 0.6) mode = 'close';
		else if (roll < 0.85) mode = 'wide';
		else if (roll < 0.95) mode = 'common';
		else mode = 'save';

		// Nudge the first few plays toward variety without making the result feel scripted.
		if (roundNumber === 1 && mode === 'save') mode = 'close';
		if (roundNumber === 2 && mode === 'spot') mode = 'wide';

		let point: TapPoint;
		let saved = false;
		if (mode === 'save') {
			point = { ...tap };
			saved = true;
		} else if (mode === 'common') {
			const left = Math.random() < 0.5;
			point = { x: left ? size.width * 0.2 : size.width * 0.8, y: size.height * 0.68 };
		} else {
			const ranges = {
				spot: [8, 38],
				close: [44, 116],
				wide: [126, 218]
			} as const;
			const [min, max] = ranges[mode];
			point = offsetFromTap(tap, min, max, size);
		}

		const outside = !insideGoal(point, size);
		const distance = Math.hypot(tap.x - point.x, tap.y - point.y);

		if (saved) {
			return { ...point, label: 'SAVED', points: '', muted: true, saved: true };
		}
		if (outside) {
			return { ...point, label: 'MISSED THE GOAL', points: '', muted: true, saved: false };
		}
		if (distance < 40) {
			return { ...point, label: 'SPOT ON', points: '+500', muted: false, saved: false };
		}
		if (distance < 100) {
			return { ...point, label: 'CLOSE', points: '+200', muted: false, saved: false };
		}
		return { ...point, label: 'WIDE', points: '+50', muted: true, saved: false };
	}

	function offsetFromTap(tap: TapPoint, min: number, max: number, size: { width: number; height: number }) {
		for (let i = 0; i < 12; i += 1) {
			const angle = Math.random() * Math.PI * 2;
			const distance = min + Math.random() * (max - min);
			const point = {
				x: tap.x + Math.cos(angle) * distance,
				y: tap.y + Math.sin(angle) * distance
			};
			if (point.x > -16 && point.x < size.width + 16 && point.y > -16 && point.y < size.height + 20) {
				return point;
			}
		}
		return {
			x: clamp(tap.x + min, -10, size.width + 10),
			y: clamp(tap.y - min * 0.55, -10, size.height + 10)
		};
	}

	function insideGoal(point: TapPoint, size: { width: number; height: number }) {
		return point.x >= size.width * 0.1 && point.x <= size.width * 0.9 && point.y >= size.height * 0.08 && point.y <= size.height * 0.78;
	}

	function clamp(value: number, min: number, max: number) {
		return Math.min(max, Math.max(min, value));
	}

	onDestroy(clearTimers);
</script>

<svelte:head>
	<meta name="description" content="Penalty King — mobile penalty prediction prototype." />
</svelte:head>

<main class="shell" class:playing={gameState !== 'idle'}>
	<div class="vignette" aria-hidden="true"></div>
	<div class="flash" class:active={flash} aria-hidden="true"></div>

	<Banner active={gameState === 'trigger'} match={selectedMatch} />

	{#if gameState !== 'idle' && gameState !== 'trigger'}
		<div class="compact-header">
			<span class="live-dot" aria-hidden="true"></span>
			<span>PENALTY — {selectedMatch}</span>
		</div>
	{/if}

	<section class="stage" aria-label="Penalty King">
		{#if gameState === 'idle'}
			<div class="idle-card">
				<h1 class="wordmark font-display">Penalty King</h1>
				<button class="next-button font-ui" onclick={startRound}>Next penalty</button>
			</div>
		{:else if isGoalVisible}
			<div class="play-card">
				<div class="timer" aria-hidden="true">
					<span style={`transform: scaleX(${progress});`}></span>
				</div>

				<div bind:this={goalEl} class="goal-measure">
					<GoalFrame interactive={isInteractive} onTap={handleTap}>
						<Reticle
							visible={!!tapPoint}
							x={tapPoint?.x ?? 0}
							y={tapPoint?.y ?? 0}
							dimmed={gameState === 'lockout' || gameState === 'resolution'}
						/>
						{#if gameState === 'resolution' && resolution}
							<Ball
								active={true}
								fromX={goalSize.width / 2}
								fromY={goalSize.height + 44}
								x={resolution.x}
								y={resolution.y}
								saved={resolution.saved}
							/>
						{/if}
					</GoalFrame>
				</div>

				<div class="instruction font-ui" class:locked={gameState === 'lockout' || gameState === 'resolution'}>
					{gameState === 'aim' ? 'TAP TO PLACE' : 'LOCKED'}
				</div>

				<div class="result-space">
					{#if gameState === 'resolution' && resolution}
						<ResultDisplay result={resolution.label} points={resolution.points} muted={resolution.muted} />
					{/if}
				</div>
			</div>
		{/if}
	</section>
</main>

<style>
	.shell {
		position: relative;
		isolation: isolate;
		display: grid;
		min-height: 100svh;
		place-items: center;
		overflow: hidden;
		background: #0a0a0c;
		color: #f5f5f0;
	}

	.vignette {
		position: fixed;
		inset: 0;
		z-index: -1;
		background:
			radial-gradient(circle at 50% 42%, rgba(245, 245, 240, 0.035), transparent 34%),
			radial-gradient(circle at 50% 115%, rgba(201, 169, 97, 0.075), transparent 38%),
			#0a0a0c;
	}

	.flash {
		position: fixed;
		inset: 0;
		z-index: 40;
		pointer-events: none;
		background: rgba(255, 255, 255, 0.04);
		opacity: 0;
		transition: opacity 110ms ease-out;
	}

	.flash.active {
		opacity: 1;
		transition-duration: 50ms;
	}

	.stage {
		display: grid;
		width: 100%;
		min-height: 100svh;
		place-items: center;
		padding: max(24px, env(safe-area-inset-top)) 16px max(24px, env(safe-area-inset-bottom));
	}

	.idle-card,
	.play-card {
		display: flex;
		width: 100%;
		max-width: 430px;
		align-items: center;
		justify-content: center;
	}

	.idle-card {
		min-height: 72svh;
		flex-direction: column;
		gap: 2.1rem;
		animation: settle 620ms ease-out both;
	}

	.wordmark {
		margin: 0;
		color: #c9a961;
		font-size: clamp(3.2rem, 16vw, 5.6rem);
		font-weight: 300;
		line-height: 0.9;
		letter-spacing: 0.015em;
	}

	.next-button {
		border: 1px solid rgba(201, 169, 97, 0.66);
		border-radius: 999px;
		background: transparent;
		color: #f5f5f0;
		padding: 0.9rem 1.35rem;
		font-size: 0.78rem;
		font-weight: 300;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		transition:
			border-color 180ms ease,
			background 180ms ease,
			transform 180ms ease;
	}

	.next-button:active {
		transform: scale(0.985);
		background: rgba(201, 169, 97, 0.06);
		border-color: rgba(201, 169, 97, 0.92);
	}

	.play-card {
		flex-direction: column;
		gap: 1.2rem;
		animation: settle 520ms ease-out both;
	}

	.compact-header {
		position: fixed;
		top: max(18px, env(safe-area-inset-top));
		left: 50%;
		z-index: 18;
		display: flex;
		align-items: center;
		gap: 0.46rem;
		color: rgba(245, 245, 240, 0.76);
		font-family: 'Inter', system-ui, sans-serif;
		font-size: 0.66rem;
		font-weight: 300;
		letter-spacing: 0.13em;
		text-transform: uppercase;
		transform: translateX(-50%);
		white-space: nowrap;
		opacity: 0;
		animation: headerIn 360ms ease-out both;
	}

	.live-dot {
		width: 6px;
		height: 6px;
		border-radius: 50%;
		background: #d63838;
		box-shadow: 0 0 0 0 rgba(214, 56, 56, 0.42);
		animation: pulse 1.15s ease-out infinite;
	}

	.timer {
		width: min(calc(100vw - 56px), 336px);
		height: 2px;
		overflow: hidden;
		background: rgba(245, 245, 240, 0.1);
	}

	.timer span {
		display: block;
		width: 100%;
		height: 100%;
		background: #c9a961;
		transform-origin: left center;
		will-change: transform;
	}

	.goal-measure {
		position: relative;
	}

	.instruction {
		min-height: 16px;
		color: rgba(245, 245, 240, 0.58);
		font-size: 0.66rem;
		font-weight: 300;
		letter-spacing: 0.26em;
		text-transform: uppercase;
		transition: color 220ms ease;
	}

	.instruction.locked {
		color: rgba(201, 169, 97, 0.72);
	}

	.result-space {
		display: grid;
		min-height: 98px;
		place-items: center;
	}

	@keyframes settle {
		from {
			opacity: 0;
			transform: translateY(8px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@keyframes headerIn {
		from {
			opacity: 0;
			transform: translate(-50%, -6px);
		}
		to {
			opacity: 1;
			transform: translate(-50%, 0);
		}
	}

	@keyframes pulse {
		0% {
			box-shadow: 0 0 0 0 rgba(214, 56, 56, 0.42);
		}
		70% {
			box-shadow: 0 0 0 8px rgba(214, 56, 56, 0);
		}
		100% {
			box-shadow: 0 0 0 0 rgba(214, 56, 56, 0);
		}
	}

	@media (max-height: 650px) {
		.play-card {
			gap: 0.75rem;
		}

		.result-space {
			min-height: 76px;
		}
	}
</style>
