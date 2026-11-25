<script>
	/**
	 * Profile — filled person from alpha-ui/icons/profile.svg (currentColor).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 18, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg viewBox="0 0 16 20" width={size} height={size} fill="none" aria-hidden="true">
		<path
			d="M16 16.2353C16 18.3145 12.4183 20 8 20C3.58172 20 0 18.3145 0 16.2353C0 14.1561 3.58172 12.4706 8 12.4706C12.4183 12.4706 16 14.1561 16 16.2353Z"
			fill="currentColor"
		/>
		<path
			d="M12.8 4.70588C12.8 7.30487 10.651 9.41177 8 9.41177C5.34903 9.41177 3.2 7.30487 3.2 4.70588C3.2 2.1069 5.34903 0 8 0C10.651 0 12.8 2.1069 12.8 4.70588Z"
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
