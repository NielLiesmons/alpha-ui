<script>
	/**
	 * Shared modal / sheet top chrome:
	 * close (X) | centered title | trailing (more ⋯ or empty spacer for balance).
	 * Desktop: 28 glass buttons, 14px title. Mobile: 44 round buttons, 17px page-title style.
	 * Comment sheets keep a local topbar (Drag grabber) but match these metrics.
	 */
	import ChromeActionButton from './ChromeActionButton.svelte';
	import Close from './icons/Close.svelte';

	/**
	 * @type {{
	 *   title?: string,
	 *   trailing?: 'more' | 'none',
	 *   closeLabel?: string,
	 *   moreLabel?: string,
	 *   onclose?: () => void,
	 *   onmore?: () => void,
	 *   class?: string
	 * }}
	 */
	let {
		title = '',
		trailing = 'none',
		closeLabel = 'Close',
		moreLabel = 'More',
		onclose = () => {},
		onmore = undefined,
		class: className = ''
	} = $props();
</script>

<div class="sheet-top {className}" aria-hidden="false">
	<ChromeActionButton variant="icon" label={closeLabel} class="top-btn close" onclick={() => onclose()}>
		<Close size={18} />
	</ChromeActionButton>
	<p class="sheet-title">{title}</p>
	{#if trailing === 'more'}
		<ChromeActionButton
			variant="more"
			label={moreLabel}
			class="top-btn more"
			onclick={() => onmore?.()}
		/>
	{:else}
		<span class="top-spacer" aria-hidden="true"></span>
	{/if}
</div>

<style>
	.sheet-top {
		position: relative;
		z-index: 5;
		flex-shrink: 0;
		/* 10 + 28btn + 10 — top pad matches horizontal pad of the button row. */
		height: 3rem;
		display: grid;
		grid-template-columns: 28px 1fr 28px;
		align-items: center;
		gap: 0.5rem;
		padding: 0.625rem;
		box-sizing: border-box;
		pointer-events: none;
	}

	.sheet-title {
		margin: 0;
		min-width: 0;
		text-align: center;
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.875rem;
		font-weight: 600;
		letter-spacing: -0.01em;
		line-height: 1.2;
		color: var(--fg-66, var(--ink66));
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		pointer-events: none;
	}

	.sheet-top :global(.chrome-btn) {
		pointer-events: auto;
	}

	.top-spacer {
		width: 1.75rem;
		height: 1.75rem;
		pointer-events: none;
	}

	@media (max-width: 767px) {
		.sheet-top {
			/* 12 + 44btn + 12 — concentric with --radius-hero (34 = 12+22). */
			height: 68px;
			grid-template-columns: 44px 1fr 44px;
			padding: 0.75rem;
		}

		.sheet-title {
			/* Match ChromeMobileBar / page topbar titles. */
			font-size: 1.0625rem;
			line-height: 1.15;
		}

		.top-spacer {
			width: 2.75rem;
			height: 2.75rem;
		}
	}
</style>
