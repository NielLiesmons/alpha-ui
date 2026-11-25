<script>
	/**
	 * Shared top-bar glass action button (Admin + Player).
	 * Desktop 28×28 radius 10px; mobile 44 round. Glass + --glass-shine.
	 * @type {{
	 *   variant?: 'more' | 'icon',
	 *   label?: string,
	 *   expanded?: boolean,
	 *   onclick?: (e: MouseEvent) => void,
	 *   onpointerdown?: (e: PointerEvent) => void,
	 *   class?: string,
	 *   children?: import('svelte').Snippet
	 * }}
	 */
	let {
		variant = 'more',
		label = 'More',
		expanded = false,
		onclick = () => {},
		onpointerdown = undefined,
		class: className = '',
		children
	} = $props();
</script>

<button
	type="button"
	class="chrome-btn {variant === 'more' ? 'more-btn' : ''} {className}"
	aria-label={label}
	aria-expanded={expanded}
	aria-haspopup={variant === 'more' ? 'menu' : undefined}
	{onclick}
	{onpointerdown}
>
	{#if children}
		{@render children()}
	{:else if variant === 'more'}
		<svg width="18" height="18" viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<circle cx="9" cy="4.5" r="1.35" fill="currentColor" />
			<circle cx="9" cy="9" r="1.35" fill="currentColor" />
			<circle cx="9" cy="13.5" r="1.35" fill="currentColor" />
		</svg>
	{/if}
</button>

<style>
	.chrome-btn {
		width: var(--chrome-desk, 1.75rem);
		height: var(--chrome-desk, 1.75rem);
		display: grid;
		place-items: center;
		padding: 0;
		border: 1px solid transparent;
		border-radius: var(--chrome-radius-desk, 0.625rem);
		background:
			linear-gradient(var(--glass), var(--glass)) padding-box,
			var(--glass-shine) border-box;
		color: var(--fg-66);
		cursor: pointer;
		box-sizing: border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
		-webkit-tap-highlight-color: transparent;
		transition: transform 0.15s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		flex-shrink: 0;
	}

	/* Scale only — match page topbar (UiShell); no hover color wash. */
	.chrome-btn:hover,
	.chrome-btn[aria-expanded='true'] {
		transform: scale(1.06);
	}

	.chrome-btn:active {
		transform: scale(0.96);
	}

	.chrome-btn :global(svg) {
		display: block;
		width: var(--chrome-icon-desk, 1rem);
		height: var(--chrome-icon-desk, 1rem);
	}

	/* Close (X) owns its size — see icons/Close.svelte (+ mobile --close-icon-size-mobile). */
	.chrome-btn :global(.close-icon),
	.chrome-btn :global(.close-icon svg) {
		width: var(--close-size, 1.125rem);
		height: var(--close-size, 1.125rem);
	}

	/* ⋮ inside the same glass hit target — keep compact (was briefly oversized). */
	.chrome-btn.more-btn :global(svg) {
		width: var(--chrome-icon-desk, 1rem);
		height: var(--chrome-icon-desk, 1rem);
	}

	@media (max-width: 767px) {
		.chrome-btn {
			width: var(--chrome-mobile, 2.75rem);
			height: var(--chrome-mobile, 2.75rem);
			border-radius: 9999px;
		}

		.chrome-btn:hover,
		.chrome-btn[aria-expanded='true'] {
			transform: scale(1.04);
		}

		.chrome-btn :global(svg) {
			width: var(--chrome-icon-md, 1.25rem);
			height: var(--chrome-icon-md, 1.25rem);
		}

		.chrome-btn.more-btn :global(svg) {
			width: var(--chrome-icon-md, 1.25rem);
			height: var(--chrome-icon-md, 1.25rem);
		}

		.chrome-btn :global(.close-icon),
		.chrome-btn :global(.close-icon svg) {
			width: var(--close-icon-size-mobile, 1.6875rem);
			height: var(--close-icon-size-mobile, 1.6875rem);
		}
	}
</style>
