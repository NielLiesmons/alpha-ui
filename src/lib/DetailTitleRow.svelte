<script>
	/**
	 * Shared detail-entity title row (Notes / Tasks / Mail / Projects).
	 * Owns spacing + title typography; apps supply leading/trailing chrome and title content.
	 * Apply class `detail-title` on the title element (h1 / textarea / span).
	 *
	 * @type {{
	 *   class?: string,
	 *   leading?: import('svelte').Snippet,
	 *   trailing?: import('svelte').Snippet,
	 *   children?: import('svelte').Snippet
	 * }}
	 */
	let { class: className = '', leading, trailing, children } = $props();
</script>

<div class="detail-title-row {className}">
	{#if leading}
		<span class="detail-title-side leading">{@render leading()}</span>
	{/if}
	<div class="detail-title-main">
		{@render children?.()}
	</div>
	{#if trailing}
		<span class="detail-title-side trailing">{@render trailing()}</span>
	{/if}
</div>

<style>
	/*
	 * Tokens (with fallbacks): --detail-title-row-pad-y-start / -mobile / -end,
	 * --detail-title-size, --detail-title-line-height.
	 * Body scroll clearance under ChromeMobileBar: --detail-scroll-pad-top(-mobile).
	 */
	.detail-title-row {
		display: flex;
		align-items: flex-start;
		gap: 10px;
		min-width: 0;
		margin: 0;
		padding: var(--detail-title-row-pad-y-start, 4px) 0
			var(--detail-title-row-pad-y-end, 10px);
		box-sizing: border-box;
	}

	@media (max-width: 767px) {
		.detail-title-row {
			padding-top: var(--detail-title-row-pad-y-start-mobile, 10px);
		}
	}

	.detail-title-side {
		flex-shrink: 0;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		line-height: 0;
		/* Optical center with first title line (~1.55rem / 1.2) */
		padding-top: 4px;
	}

	/* Snippet passed but rendered nothing — drop the flex gap slot */
	.detail-title-side:empty {
		display: none;
	}

	.detail-title-main {
		flex: 1 1 auto;
		min-width: 0;
		display: flex;
		flex-wrap: wrap;
		align-items: baseline;
		column-gap: 0;
		row-gap: 0;
	}

	.detail-title-row :global(.detail-title) {
		flex: 1 1 auto;
		min-width: 0;
		max-width: 100%;
		margin: 0;
		padding: 0;
		border: 0;
		outline: none;
		resize: none;
		background: transparent;
		font-family: var(--font);
		font-size: var(--detail-title-size, 1.55rem);
		font-weight: 700;
		letter-spacing: -0.03em;
		line-height: var(--detail-title-line-height, 1.2);
		color: var(--fg);
		overflow-wrap: anywhere;
		word-break: break-word;
	}

	.detail-title-row :global(h1.detail-title) {
		white-space: normal;
	}

	.detail-title-row :global(textarea.detail-title) {
		/* Hide scrollbar; height comes from hugTitle so wrapped lines stay visible */
		overflow: hidden;
		width: 100%;
		white-space: pre-wrap;
	}

	.detail-title-row :global(.detail-title::placeholder) {
		color: var(--fg-33);
		font-weight: 600;
	}
</style>
