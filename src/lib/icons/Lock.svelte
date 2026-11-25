<script>
	/**
	 * Lock — fill path from alpha-ui/icons/lock.svg (currentColor).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 16, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{((Math.round((size * 10) / 8)) / 16)}rem">
	<svg
		viewBox="0 0 8 10"
		width={size}
		height={(size * 10) / 8}
		fill="none"
		aria-hidden="true"
	>
		<path
			fill-rule="evenodd"
			clip-rule="evenodd"
			d="M4 0C5.3785 0 6.5 1.1215 6.5 2.5V4H7C7.5515 4 8 4.4485 8 5V9C8 9.5515 7.5515 10 7 10H1C0.4485 10 0 9.5515 0 9V5C0 4.4485 0.4485 4 1 4H1.5V2.5C1.5 1.1215 2.6215 0 4 0ZM4 6C3.44772 6 3 6.44772 3 7C3 7.55228 3.44772 8 4 8C4.55228 8 5 7.55228 5 7C5 6.44772 4.55228 6 4 6ZM4 1C3.173 1 2.5 1.673 2.5 2.5V4H5.5V2.5C5.5 1.673 4.827 1 4 1Z"
			fill="currentColor"
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
