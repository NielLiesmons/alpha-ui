<script>
	/**
	 * Soft metaball loader — two blobs always fused, alternating size.
	 * Color inherits `currentColor` (chrome: set quiet ink via parent).
	 * @type {{
	 *   size?: 'sm' | 'default' | number,
	 *   label?: string,
	 *   class?: string
	 * }}
	 */
	let { size = 'default', label = 'Loading', class: className = '' } = $props();

	const uid = `mb-${Math.random().toString(36).slice(2, 9)}`;

	/** @param {'sm' | 'default' | number} s */
	function dims(s) {
		if (typeof s === 'number' && Number.isFinite(s) && s > 0) {
			const h = Math.max(8, Math.round(s));
			return { w: Math.round(h * 1.75), h };
		}
		if (s === 'sm') return { w: 18, h: 10 };
		return { w: 28, h: 16 };
	}

	const box = $derived(dims(size));
</script>

<span
	class="metaball {className}"
	style="width:{box.w}px;height:{box.h}px"
	role="status"
	aria-label={label}
>
	<svg
		class="metaball-svg"
		width={box.w}
		height={box.h}
		viewBox="0 0 28 16"
		aria-hidden="true"
		focusable="false"
	>
		<defs>
			<filter
				id="{uid}-goo"
				x="-40%"
				y="-40%"
				width="180%"
				height="180%"
				color-interpolation-filters="sRGB"
			>
				<feGaussianBlur in="SourceGraphic" stdDeviation="2.2" result="blur" />
				<feColorMatrix
					in="blur"
					mode="matrix"
					values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 18 -7"
					result="goo"
				/>
				<feComposite in="SourceGraphic" in2="goo" operator="atop" />
			</filter>
		</defs>
		<g filter="url(#{uid}-goo)" fill="currentColor">
			<circle class="ball left" cx="9" cy="8" r="5.2" />
			<circle class="ball right" cx="19" cy="8" r="5.2" />
		</g>
	</svg>
</span>

<style>
	.metaball {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
		line-height: 0;
		color: inherit;
		vertical-align: middle;
	}

	.metaball-svg {
		display: block;
		overflow: visible;
	}

	.ball {
		transform-origin: center;
		transform-box: fill-box;
	}

	.ball.left {
		animation: mb-left 1.15s ease-in-out infinite;
	}

	.ball.right {
		animation: mb-right 1.15s ease-in-out infinite;
	}

	@keyframes mb-left {
		0%,
		100% {
			transform: scale(1.22);
		}
		50% {
			transform: scale(0.72);
		}
	}

	@keyframes mb-right {
		0%,
		100% {
			transform: scale(0.72);
		}
		50% {
			transform: scale(1.22);
		}
	}

	@media (prefers-reduced-motion: reduce) {
		.ball.left,
		.ball.right {
			animation: mb-pulse 1.4s ease-in-out infinite;
		}

		@keyframes mb-pulse {
			0%,
			100% {
				opacity: 0.55;
				transform: scale(0.95);
			}
			50% {
				opacity: 1;
				transform: scale(1.05);
			}
		}
	}
</style>
