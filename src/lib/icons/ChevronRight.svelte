<script>
	/**
	 * ChevronRight — default leading glyph for popover rows without a dedicated icon.
	 * @type {{ size?: number, className?: string, strokeWidth?: number | string }}
	 */
	let { size = 14, className = '', strokeWidth = 1.5 } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg
		viewBox="0 0 8 14"
		width={size}
		height={size}
		fill="none"
		aria-hidden="true"
	>
		<path
			d="M0.7 0.7 L6.7 6.7 L0.7 12.7"
			stroke="currentColor"
			stroke-width={strokeWidth}
			stroke-linecap="round"
			stroke-linejoin="round"
		/>
	</svg>
</span>

<style>
	.icon {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
		line-height: 0;
		color: inherit;
	}

	.icon :global(svg) {
		display: block;
		width: 100%;
		height: 100%;
		overflow: visible;
	}

	.icon :global(path) {
		vector-effect: non-scaling-stroke;
		stroke-width: var(--icon-stroke, 1.5);
	}
</style>
