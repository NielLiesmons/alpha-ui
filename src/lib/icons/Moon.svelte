<script>
	/**
	 * Moon — filled (theme Dark).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 16, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg viewBox="0 0 20 20" width={size} height={size} fill="none" aria-hidden="true">
		<path
			fill="currentColor"
			d="M17.293 13.293A8 8 0 0 1 6.707 2.707a8.001 8.001 0 1 0 10.586 10.586Z"
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
</style>
