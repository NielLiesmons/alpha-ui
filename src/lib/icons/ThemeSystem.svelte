<script>
	/**
	 * Theme system — filled display (follow OS light/dark).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 16, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg viewBox="0 0 20 20" width={size} height={size} fill="none" aria-hidden="true">
		<path
			fill="currentColor"
			fill-rule="evenodd"
			clip-rule="evenodd"
			d="M3.5 4.75A2.75 2.75 0 0 1 6.25 2h7.5A2.75 2.75 0 0 1 16.5 4.75v7.5a2.75 2.75 0 0 1-2.75 2.75h-2.19l.47 1.25h1.72a.75.75 0 0 1 0 1.5H6.25a.75.75 0 0 1 0-1.5h1.72l.47-1.25H6.25A2.75 2.75 0 0 1 3.5 12.25v-7.5Zm1.5 0c0-.69.56-1.25 1.25-1.25h7.5c.69 0 1.25.56 1.25 1.25v7.5c0 .69-.56 1.25-1.25 1.25h-7.5c-.69 0-1.25-.56-1.25-1.25v-7.5Z"
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
