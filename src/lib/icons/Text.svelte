<script>
	/**
	 * Text — stroke path from alpha-ui/icons/text.svg (currentColor).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 15, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg viewBox="0 0 23 22" width={size} height={size} fill="none" aria-hidden="true">
		<path
			d="M0.5 5.5V2.5C0.5 1.39543 1.39543 0.5 2.5 0.5H11.5M11.5 0.5H20.5C21.6046 0.5 22.5 1.39543 22.5 2.5V5.5M11.5 0.5V21.5M11.5 21.5H7.5M11.5 21.5H15.5"
			stroke="currentColor"
			stroke-linecap="round"
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
		stroke-width: 1.5;
	}
</style>
