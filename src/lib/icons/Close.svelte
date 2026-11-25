<script>
	/**
	 * Dismiss / close X.
	 * Desktop: `size` (default 18). Mobile: shared ~50% bump via `--close-icon-size-mobile` (27).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 18, className = '' } = $props();
</script>

<span class="icon close-icon {className}" style:--close-size="{size}px">
	<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
		<path
			d="M4.5 4.5 13.5 13.5M13.5 4.5 4.5 13.5"
			stroke="currentColor"
			stroke-width="1.7"
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
		width: var(--close-size, 1.125rem);
		height: var(--close-size, 1.125rem);
	}
	.icon :global(svg) {
		display: block;
		width: 100%;
		height: 100%;
	}

	/* One shared mobile rule — SheetTopbar, comments, lightbox, voice player, etc. */
	@media (max-width: 767px) {
		.icon {
			width: var(--close-icon-size-mobile, 1.6875rem);
			height: var(--close-icon-size-mobile, 1.6875rem);
		}
		/* Beat ChromeActionButton / local .chrome-btn svg size clamps */
		.icon :global(svg) {
			width: 100% !important;
			height: 100% !important;
		}
	}
</style>
